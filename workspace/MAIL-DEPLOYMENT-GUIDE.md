# Mail Server Deployment - Quick Start Guide

## Overview

You have a complete mail server deployment package. Here's what each script does:

### Main Scripts

1. **mail-server-deploy.sh** — Full OS setup + Postfix + Dovecot + PostfixAdmin
   - Runs as root
   - Takes ~5-10 minutes
   - Creates database, installs all services, configures SSL

2. **mail-server-bulk-import.py** — Imports your CSVs into the database
   - Runs as root: `sudo python3 mail-server-bulk-import.py <csv_dir>`
   - Reads: domains.csv, users.csv, aliases.csv, sieve-scripts/
   - Creates maildir directories with proper permissions
   - Populates SQLite database via parameterized SQL
   - **Replaces the old .sh version**, which broke on apostrophes (xargs) and
     quoted multi-destination aliases (IFS split). Python csv + sqlite3 handles
     both. Stdlib-only, no venv needed. Guards against an unbuilt schema.

3. **mail-server-test.sh** — Validates everything works
   - Run as regular user
   - Tests all services, ports, database, SSL, mail delivery

4. **mail-server-backup.sh** — Daily backups and restore
   - Usage: `sudo bash mail-server-backup.sh backup /backup/path`
   - Usage: `sudo bash mail-server-restore.sh restore /backup/path/mail-backup-*.tar.gz`

5. **mail-server-monitor.sh** — Health checks (run via cron)
   - Usage: `bash mail-server-monitor.sh admin@example.com`
   - Checks disk, queue, SSL expiry, database integrity
   - Sends alerts on issues

6. **mail-server-migrate.sh** — Migrates mail from old server (imapsync)
   - Usage: `bash mail-server-migrate.sh old-mail.server.com username password`
   - Runs incremental syncs with retry logic

7. **ldif-to-postfixadmin.py** — Converts OMS LDIF to CSVs
   - Usage: `python3 ldif-to-postfixadmin.py export.ldif output_dir/`

8. **mail-server-roundcube.sh** — Roundcube webmail (1.6.x)
   - Usage: `sudo bash mail-server-roundcube.sh webmail.padz.net admin@padz.net`
   - SQLite prefs DB, nginx vhost, IMAP/SMTP/ManageSieve wiring to local Dovecot/Postfix
   - Users log in with full email + mail password; authenticates against Dovecot IMAP
   - Includes managesieve plugin so users can edit the Sieve rules migrated from OMS
   - VERIFY on box: config key names (`imap_host`/`smtp_host`) against shipped `config/defaults.inc.php`

---

## Deployment Flow

### Day 1: VPS Provisioning & Setup

1. **Receive VPS IP from Contabo**
   ```bash
   ssh root@<VPS_IP>
   ```

2. **Upload deployment scripts**
   ```bash
   scp mail-server-*.sh root@<VPS_IP>:/root/
   scp ldif-to-postfixadmin.py root@<VPS_IP>:/root/
   ```

3. **Run main deployment** (takes ~10 min)
   ```bash
   ssh root@<VPS_IP>
   bash mail-server-deploy.sh mail.padz.net admin@padz.net
   ```

4. **Update DNS MX record** (point to new VPS IP, lower TTL to 5 min)

### Day 2: Import Data & Test

1. **Upload your CSVs and Sieve scripts to VPS**
   ```bash
   scp -r z/ root@<VPS_IP>:/root/csv-import/
   ```

2. **Import all data**
   ```bash
   ssh root@<VPS_IP>
   bash mail-server-bulk-import.sh /root/csv-import
   ```

3. **Run tests**
   ```bash
   bash mail-server-test.sh mail.padz.net
   ```

4. **Final mail cutover** (MX record now live)

### Day 3+: Migration & Verification

1. **Run incremental sync from OMS** (first time, syncs all mail)
   ```bash
   bash mail-server-migrate.sh old-mail.padz.net djpadz <password>
   ```

2. **Verify user mailboxes**
   - Have users log in with their mail client
   - Check folder counts match expectations
   - Verify Sieve filters are active

3. **Set up automated backups** (cron)
   ```bash
   # Add to crontab
   0 2 * * * /root/mail-server-backup.sh backup /backups
   0 * * * * /root/mail-server-monitor.sh admin@padz.net
   ```

4. **Decommission OMS** (after 48-72 hours verification)

---

## Key Configuration

All scripts use these defaults:

| Setting | Value |
|---------|-------|
| Mail user | vmail (UID 5000) |
| Mail home | /var/mail/vhosts |
| Database | SQLite at /var/lib/postfixadmin/postfixadmin.db |
| SSL | Let's Encrypt (auto-renews) |
| SMTP | Port 25 (inbound), 587 (submission) |
| IMAP | Port 993 (SSL), 143 (starttls) |
| POP3 | Port 995 (SSL), 110 (starttls) |
| Sieve | Port 4190 (ManageSieve) |
| PostfixAdmin | HTTPS on port 443 |

---

## Common Tasks

### Add a new domain/user via CLI

```bash
# Add domain
sqlite3 /var/lib/postfixadmin/postfixadmin.db \
  "INSERT INTO domain (domain, active) VALUES ('newdomain.com', 1);"

# Add user
sqlite3 /var/lib/postfixadmin/postfixadmin.db \
  "INSERT INTO mailbox (username, local_part, domain, password, maildir, active) \
   VALUES ('user@newdomain.com', 'user', 'newdomain.com', '{SSHA}...hash...', '/var/mail/vhosts/newdomain.com/user', 1);"

# Create maildir
mkdir -p /var/mail/vhosts/newdomain.com/user/{cur,new,tmp}
chown -R 5000:5000 /var/mail/vhosts/newdomain.com/user
chmod 750 /var/mail/vhosts/newdomain.com/user
```

### Check mail queue

```bash
postqueue -p          # List pending mail
postsuper -d ALL      # Delete all queued mail (use with caution)
postfix flush         # Force immediate delivery attempt
```

### Monitor real-time activity

```bash
# Watch Postfix logs
tail -f /var/log/mail.log | grep postfix

# Watch Dovecot logs
tail -f /var/log/mail.log | grep dovecot

# Check IMAP/POP3 connections
netstat -an | grep ESTABLISHED | grep -E ':143|:993|:995'
```

### Verify Sieve scripts are working

```bash
# Check if user has sieve scripts
ls -la /var/mail/vhosts/<domain>/<user>/sieve/

# Test Sieve syntax
sievec /var/mail/vhosts/<domain>/<user>/sieve/rule-1.sieve
```

---

## Troubleshooting

### Postfix won't start

```bash
# Check config
postfix check

# View logs
journalctl -u postfix -n 50

# Common issue: port already in use
netstat -tlnp | grep 25
```

### Dovecot won't start

```bash
# Check config
doveconf

# View logs
journalctl -u dovecot -n 50

# Common issue: missing maildir
ls -la /var/mail/vhosts/
```

### Mail not delivering

```bash
# Check if domain/user exists in database
sqlite3 /var/lib/postfixadmin/postfixadmin.db \
  "SELECT * FROM mailbox WHERE username='user@domain.com';"

# Check maildir exists and has correct permissions
ls -la /var/mail/vhosts/<domain>/<user>

# Check Postfix logs
tail -f /var/log/mail.log | grep user@domain.com
```

### IMAP login fails

```bash
# Test with telnet (port 143, then STARTTLS)
telnet localhost 143
> STARTTLS
> LOGIN user@domain.com password

# Check Dovecot auth
journalctl -u dovecot -n 50 | grep -i "auth\|login"

# Verify user in database
sqlite3 /var/lib/postfixadmin/postfixadmin.db \
  "SELECT username, password FROM mailbox WHERE username='user@domain.com';"
```

### Sieve scripts not running

```bash
# Verify sieve is enabled in Dovecot
doveconf | grep sieve

# Check script syntax
sievec /var/mail/vhosts/<domain>/<user>/sieve/rule-1.sieve

# Check Dovecot logs
journalctl -u dovecot -n 50 | grep sieve
```

### SSL certificate issues

```bash
# Check certificate
openssl x509 -text -noout -in /etc/letsencrypt/live/mail.padz.net/fullchain.pem

# Check expiry
openssl x509 -enddate -noout -in /etc/letsencrypt/live/mail.padz.net/fullchain.pem

# Manual renewal
certbot renew --force-renewal

# Reload services after cert update
systemctl restart postfix dovecot nginx
```

---

## Performance Tuning (Optional)

### For high-volume servers

**Dovecot** (`/etc/dovecot/conf.d/10-master.conf`):
```
service imap-login {
  process_min_avail = 4
  process_limit = 512
}
```

**Postfix** (`/etc/postfix/main.cf`):
```
max_idle = 900
max_use = 100
default_process_limit = 100
smtp_destination_concurrency_limit = 20
```

**SQLite** (for high query rates):
```
# Add to queries
PRAGMA journal_mode=WAL;
PRAGMA synchronous=NORMAL;
```

---

## Security Hardening

### Firewall (ufw)

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp       # SSH
ufw allow 25/tcp       # SMTP
ufw allow 587/tcp      # Submission
ufw allow 143/tcp      # IMAP
ufw allow 993/tcp      # IMAPS
ufw allow 995/tcp      # POP3S
ufw allow 443/tcp      # HTTPS (PostfixAdmin)
ufw enable
```

### Fail2ban (optional)

```bash
apt-get install fail2ban
# Monitors Postfix/Dovecot logs, blocks IPs with repeated failed auth
```

### SSL/TLS Hardening

Already configured in deployment script:
- TLS 1.2+ only (no SSLv2/v3, no TLS 1.0/1.1)
- Strong ciphers only
- HSTS headers on PostfixAdmin

---

## Monitoring & Alerts

### Daily health check (add to crontab)

```bash
0 * * * * /root/mail-server-monitor.sh admin@padz.net
```

### Weekly backup (add to crontab)

```bash
0 2 * * 0 /root/mail-server-backup.sh backup /backups
```

### Check Postfix queue size

```bash
postqueue -p | tail -1
```

### Monitor disk usage

```bash
df -h /var/mail/vhosts
```

---

## Next Steps When VPS Is Ready

1. **Have your domain, CSVs, and these scripts ready**
2. **SSH into VPS as root**
3. **Copy scripts to VPS**
4. **Run `mail-server-deploy.sh`** with your domain
5. **Upload CSVs and run `mail-server-bulk-import.sh`**
6. **Run `mail-server-test.sh` to verify**
7. **Update DNS MX records**
8. **Run `mail-server-migrate.sh` to sync old mail**
9. **Set up cron jobs for backups & monitoring**

That's it. You're live.

---

Questions? Check the logs or the troubleshooting section above.
