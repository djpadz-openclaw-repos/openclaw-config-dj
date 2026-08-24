# Transparent Password Rehashing on Login

Upgrades migrated OMS hashes (MD5-CRYPT / SSHA1) to a strong scheme as each user
logs in, using the plaintext Dovecot has available post-login. Adapted from
die-welt.net's Debian-13 writeup, ported to SQLite + stdlib (no passlib/MySQLdb).

## Prerequisites
- `auth_allow_weak_schemes = yes` is set (so old hashes still authenticate)
- `auth_mechanisms = plain login` (plaintext must reach the server for rehash)
- ssl = required (so that plaintext only crosses inside TLS)

## STEP 0 — VERIFY THE HASH FORMAT FIRST (do not skip)

My script builds `{SSHA512}base64(digest + salt)`. I have NOT confirmed Dovecot
parses that exact byte order on your build. Confirm before deploying, or you'll
rehash every user into hashes Dovecot can't verify (locking them out once weak
schemes are later disabled).

```bash
# Generate a reference SSHA512 hash with Dovecot itself:
doveadm pw -s SSHA512 -p 'testpass'
# -> {SSHA512}....

# Verify a hash my script produced (paste one in):
doveadm pw -t '{SSHA512}<hash-from-script>' -p 'testpass'
# -> "verified" means the format/byte-order matches. If it fails, DO NOT deploy;
#    switch the script to shell out to `doveadm pw -s SSHA512` instead of
#    hand-rolling the hash (safest), or to ARGON2ID.
```

If `doveadm pw -t` does NOT verify my hash, use this drop-in replacement for
`make_ssha512()` that delegates to doveadm (authoritative, but spawns a process):

```python
import subprocess
def make_ssha512(plain: str) -> str:
    out = subprocess.run(
        ["doveadm", "pw", "-s", "SSHA512", "-p", plain],
        capture_output=True, text=True, check=True,
    )
    return out.stdout.strip()
```
(Note: this passes the plaintext via argv, briefly visible in the process list.
Acceptable for a login-triggered one-shot; the die-welt author flagged the same
tradeoff. For zero leak, use stdin-based hashing — but confirm your doveadm supports it.)

## STEP 1 — passdb: expose plaintext as userdb_plain_pass

Edit `/etc/dovecot/conf.d/10-sqlite.conf`, add the field to the passdb query:

```
passdb sql {
  sql_driver = sqlite
  sqlite_path = /var/lib/postfixadmin/postfixadmin.db
  query = SELECT password, '%{password}' AS userdb_plain_pass \
          FROM mailbox WHERE username = '%{user}' AND active = '1'
}
```

**SECURITY — plaintext in logs (the real risk, NOT injection):**
SQL injection is NOT a concern here: Dovecot's SQL driver auto-escapes `%{}`
expansions (verified on-box — an `ab'cd` password expands to `ab''cd` before
hitting SQL). The actual exposure is that `userdb_plain_pass` carries the
CLEARTEXT password through the auth pipeline. If auth debug logging is ever on
while this is live, plaintext passwords land in the logs. So:
- Keep `auth_verbose_passwords` and `auth_debug` / auth verbose logging OFF in prod.
- Treat this whole mechanism as TEMPORARY — it is removed entirely at STEP 6
  once all users are rehashed. Don't leave `userdb_plain_pass` in the query
  longer than the migration window needs.

## STEP 2 — add a prefetch userdb

The prefetch userdb reads fields the passdb already returned (the plain pass),
making it available to the post-login script as $PLAIN_PASS. Keep the existing
`userdb static` too — Postfix/LMTP delivery needs it to resolve the mailbox.

Add BEFORE the existing `userdb static` block:

```
userdb prefetch {
}

userdb static {
  fields {
    uid = 5000
    gid = 5000
    home = /mail/vhosts/%{user | domain}/%{user | username}
  }
}
```

## STEP 3 — post-login service

Add to `/etc/dovecot/conf.d/10-master.conf` (or a new conf.d file).

Dovecot 2.4 syntax: there is NO `protocol imap { postlogin = ... }` setting (that
was 2.3 and throws "Unknown setting: postlogin"). Instead, override the imap
service's `executable` to append a post-login socket NAME, then define a service
with a matching `unix_listener` of that same name. The link is by name match.

```
service imap {
  # append the post-login socket name; imap does a post-login lookup on it
  executable = imap imap-postlogin
}

service imap-postlogin {
  executable = script-login /usr/local/bin/mail-server-rehash-passwords.py
  user = vmail
  group = www-data
  unix_listener imap-postlogin {
  }
}
```

**CRITICAL — `group = www-data` is required, not optional.** Dovecot sets only
the service's uid + PRIMARY gid on setuid; it does NOT apply the user's
supplementary groups from /etc/group. So even after `usermod -aG www-data vmail`
(and a restart), the post-login process still can't read the www-data-owned DB
unless the group is named explicitly here. Symptom if omitted: login works but
hash never flips; script logs `OperationalError: unable to open database file`.

For POP3 users, mirror it: `service pop3 { executable = pop3 pop3-postlogin }`
plus a matching `service pop3-postlogin { ... unix_listener pop3-postlogin {} }`.

Install the script:
```bash
cp mail-server-rehash-passwords.py /usr/local/bin/
chmod 755 /usr/local/bin/mail-server-rehash-passwords.py
```

## STEP 4 — DB write access

The script runs as `vmail` and must WRITE the PostfixAdmin DB (which is owned
www-data:www-data). Options, least-surprising first:
- Add vmail to the www-data group and make the DB group-writable:
  ```bash
  usermod -aG www-data vmail
  chmod 660 /var/lib/postfixadmin/postfixadmin.db
  chmod 770 /var/lib/postfixadmin           # dir needs write for -journal/-wal
  ```
  (SQLite writes create -wal/-journal siblings, so the DIR must be group-writable too.)

## STEP 5 — restart + watch

```bash
doveconf -n >/dev/null && systemctl restart dovecot
# Log in as a test user (webmail or IMAP), then confirm the row upgraded:
sqlite3 /var/lib/postfixadmin/postfixadmin.db \
  "SELECT username, substr(password,1,10) FROM mailbox WHERE username='djpadz@padz.net';"
# Should now show {SSHA512}
```

## STEP 6 — cleanup (after ALL active users have logged in once)

```bash
# Find who's still on an old scheme:
sqlite3 /var/lib/postfixadmin/postfixadmin.db \
  "SELECT username FROM mailbox WHERE password NOT LIKE '{SSHA512}%';"
```
When that returns empty:
1. Remove the imap-postlogin service + protocol imap postlogin line
2. Remove `userdb prefetch` and the `userdb_plain_pass` field from the passdb query
3. Remove `auth_allow_weak_schemes = yes`
4. `systemctl restart dovecot`

Now you're on strong hashes only, weak schemes disabled.

## Caveats
- Only upgrades on PLAIN/LOGIN auth (plaintext needed). Fine here — that's the config.
- POP3 users: add a matching `service pop3-postlogin` + `protocol pop3 { postlogin }`.
- Webmail (Roundcube) logs in via IMAP with plaintext, so those logins trigger it too.
- Users who never log in keep their old hash; they simply won't authenticate once
  weak schemes are disabled — expected, handle via a reset for stragglers.

## ⚠️ MASTER-USER / MIGRATION INTERACTION (critical)
The PROXYAUTH migration authenticates to Dovecot as a **master user**
(`--user2 "realuser*migrate"`, see mail-server-migrate.sh + the `passwd.masterusers`
passdb in mail-server-deploy.sh). During such a login Dovecot sets `USER` to the
REAL user but the plaintext it exposes is the **master** password, not the user's.

Without a guard, this post-login script would overwrite EVERY migrated user's stored
hash with a hash of the master password on each 4-hourly sync — silent, mass
credential corruption that only surfaces at cutover when nobody can log in.

The script guards against this two ways (both in mail-server-rehash-passwords.py):
1. **Fast skip** on master-user env vars (`MASTER_USER`/`AUTH_MASTER_USER`/`ORIG_USER`).
   Names vary by build, so this alone is not trusted.
2. **Verify-before-write (the real safety net)**: before rehashing, it runs
   `doveadm pw -t <stored_hash> -p <plaintext>` and only proceeds if the plaintext
   actually matches the user's CURRENT stored hash. The master password will never
   verify against a user's weak migrated hash, so migration logins never rehash.
   Fails closed (skip) on any error.

**Do not remove either guard while the migration master user exists.** They can go
away only after both the migration is fully cut over (master passdb removed) AND
weak-scheme cleanup is done.
