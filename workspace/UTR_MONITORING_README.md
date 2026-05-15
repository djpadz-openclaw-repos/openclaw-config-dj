# Ubiquiti Travel Router Availability Monitor

## Overview
Monitors https://store.ui.com/us/en/category/wifi-special-devices/products/utr for "Sold Out" or "Add to Cart" status changes and sends Telegram alerts.

## Current Status
- **Script**: `/home/node/.openclaw/workspace/scripts/check-utr-availability.sh`
- **Main executable**: `/home/node/.openclaw/workspace/utr-monitor`
- **State file**: `/home/node/.openclaw/workspace/utr_state.json`
- **Logs**: `/home/node/.openclaw/workspace/logs/utr-monitor.log`
- **Environment**: `/home/node/.openclaw/workspace/utr-monitor.env` (secured)

## Components
1. **Main script**: Checks UI store page, determines availability
2. **State management**: Tracks last known status in JSON file
3. **Telegram integration**: Sends alerts to chat ID 8623402151
4. **Logging**: All runs logged with timestamps and results
5. **Error handling**: Timeouts (30s) and failure alerts

## Usage

### Manual Check
```bash
/home/node/.openclaw/workspace/utr-monitor
```

### Test Without Alerts
```bash
/home/node/.openclaw/workspace/test-utr-manual.sh
```

### Check Current Status
```bash
cat /home/node/.openclaw/workspace/utr_state.json
```

### View Logs
```bash
tail -f /home/node/.openclaw/workspace/logs/utr-monitor.log
```

## Scheduling (External Setup Required)
Since `crontab` is not available in this environment, schedule externally:

### Option 1: Systemd Timer (Recommended)
Create `/etc/systemd/system/utr-monitor.service`:
```ini
[Unit]
Description=UTR Availability Monitor
After=network.target

[Service]
Type=oneshot
EnvironmentFile=/home/node/.openclaw/workspace/utr-monitor.env
ExecStart=/home/node/.openclaw/workspace/utr-monitor
User=node
WorkingDirectory=/home/node/.openclaw/workspace
```

Create `/etc/systemd/system/utr-monitor.timer`:
```ini
[Unit]
Description=Run UTR monitor every 5 minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```

Enable with:
```bash
sudo systemctl enable utr-monitor.timer
sudo systemctl start utr-monitor.timer
```

### Option 2: External Cron Server
Add to crontab on another server:
```cron
*/5 * * * * ssh node@this-server /home/node/.openclaw/workspace/utr-monitor
```

### Option 3: Kubernetes CronJob
If running in Kubernetes, create a CronJob resource.

## Alert Messages
- **Available**: 🚨 *Ubiquiti Travel Router is AVAILABLE!* Check it out: [URL]
- **Sold Out**: ❌ Ubiquiti Travel Router is now SOLD OUT
- **Monitor Failed**: ⚠️ UTR monitor failed with code [code]. Check logs.

## Troubleshooting
1. **No Telegram alerts**: Check `TELEGRAM_TOKEN` in environment file
2. **Script fails**: Check logs for curl errors or page structure changes
3. **Timeout**: Network issues or UI store blocking requests
4. **State not updating**: Check permissions on state file

## Maintenance
- Logs auto-rotate (keep last 1000 lines)
- Environment file secured (chmod 600)
- All components in `/home/node/.openclaw/workspace/`
