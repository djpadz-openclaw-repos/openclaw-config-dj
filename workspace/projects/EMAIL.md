# EMAIL.md — Email Automation

Major work 2026-03-23.

## Inboxes

- `dj@heru.net` — Microsoft 365, Graph API (msal token in `ms_token_rw.json`)
- `djpadz@mac.com` — iCloud IMAP SSL port 993, credentials in 1Password (`email: djpadz@mac.com`)
- `djpadz@padz.net` — PadzNet, SSL port 993, `mail.padz.net`, credentials in 1Password (`email: djpadz@padz.net`) — do NOT use STARTTLS/143 (had connection issues)

## Cleanup Rules (run_email_cleanup.py)

- **Teams notifications** (`noreply@teams.microsoft.com`) — delete after 6h
- **Login codes** — delete after 6h (subject keywords + trusted auth senders)
- **SaneBox digests** — delete after 24h
- **Calendar responses** (Accepted/Declined/Tentative + .ics) — archive after 24h
- **USPS Informed Delivery** (`@usps.gov`) — delete after 2 days (mac.com + 365)
- **Venmo "You paid"** (`@venmo.com`, subject contains "You paid") — move to `@Receipts` after 2 days (padz.net only)
- **Receipts** — move to `@Receipts` on subject keyword match (all inboxes); no age requirement
- **rblmon no-block** (`alert@rblmon.com`, "no blocks" in subject) — delete immediately (padz.net)
- **Headway reminders** (`@e.headway.co`) — delete after appointment date passes; fallback 2 days
- **Scripps video visit** (`MyScrippsDoNotReply@myscripps.org`, "Video Visit Direct Join") — delete after 24h (padz.net)

## Notifier (notify_actionable_emails.py)

- Checks unread mail, classifies with Claude Haiku, sends Telegram alerts for actionable emails
- State in `email_notified.json` — uses Graph message IDs for 365, IMAP UIDs for `mac.com` (fixed 2026-03-23; was using unstable sequence numbers causing old emails to re-alert)
- Alert format includes: account, From, **Date**, Subject, Why
- Priority sender: Mohamed Abou Shousha — always alert immediately

## Daemon (actionable_email_daemon.py)

- Persistent systemd service: `actionable-email-daemon.service` — restart with `systemctl --user restart actionable-email-daemon.service`
- Uses IMAP IDLE for `mac.com` (SSL/993) and `padz.net` (SSL/993); Graph polling fallback for `heru.net`
- Has its own `SKIP_SENDERS` / `SKIP_SENDER_HOSTS` — must be kept in sync with `notify_actionable_emails.py`
- IronPort hosts skipped: `@mx1/mx2/mx3/sma.ctb.padz.net`

## Key Lessons

- iCloud IMAP rewrites From headers (Hide My Email aliases show as sender); search by subject not From
- iCloud IMAP search doesn't index bodies in `@SaneNews` reliably
- Graph API DELETE is hard-delete; IMAP delete moves to trash (recoverable) — Graph should be changed to move to Deleted Items to match
- **Always backtick domains in email discussions** (e.g. `venmo.com`) to suppress Telegram link previews — Dj has asked for this repeatedly
- `padz.net` IMAP uses SSL/993 — do NOT switch to STARTTLS/143, it had connection issues
- The daemon and the notifier script are separate codepaths — changes to skip lists etc. must be applied to both
