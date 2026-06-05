# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Apple Developer credentials
- Anything environment-specific

## Apple Developer

- **Team ID**: N2XUW3D6G2
- **Bundle ID Prefix**: com.famous-peers (for Famous Peers project)

Use this team ID in all xcodegen `project.yml` files:
```yaml
settings:
  DEVELOPMENT_TEAM: "N2XUW3D6G2"
```

## 2026 Toyota bZ (Daily Driver)

- **Dealer:** Norm Reeves Toyota
- **Charging:** Home Level 2 charger (overnight charging)
- **CarPlay:** Connected, but EV routing is unreliable — use standard navigation + manual charger lookups for longer trips
- **Typical use:** Local/day trips from home base; work-from-home (no commute)

## Home Tech

- **UniFi mesh network** — monitors band usage for mesh uplink (2.4 GHz vs 5 GHz)
- **Tailscale VPN** — ACL rules for exit node usage; connects to home PC when LAN blocks connections
- **DBeaver** with SOCKS proxy for remote DB access (needs `proxyDNS=true` equivalent)
- **Media server:** Sonarr (wanted to separate metadata from media volume for faster snapshots)
- **Deluge** torrent client — modified to handle magnet URI redirects from submitted URLs
- **Radio show archive:** ~dozen shows in MP3; ID3 tagging: Title=episode+date, Artist=host/station, Album=show series, Album Artist=station
- **Electronics:** u-blox NEO-6M GPS module (5-pin: VCC, GND, TX, RX, PPS), uses Fritzing for diagrams

## OpenClaw Queue Mode

- **Global queue mode** is set via `messages.queue.mode` in `~/.openclaw-dj/openclaw.json` (NOT `agents.defaults.queue`)
- Currently set to `steer` — inbound messages interrupt the current run at the next tool boundary instead of queuing up
- **Per-session override:** `/queue <mode>`; reset with `/queue reset`
- **Doc:** https://docs.openclaw.ai/concepts/queue

## Dropbox Integration

- **Upload directory:** `/Gizmo` in your Dropbox
- **CLI location:** `~/.openclaw/workspace/skills/dropbox-custom/dropbox-cli.py`
- **Usage:** `.venv/bin/python dropbox-cli.py <command> <path>`
- **Commands:** list, upload, download, delete, share
- **Auth:** Refresh token flow via 1Password (item: `Dropbox OAuth2`, vault: OpenClaw) — auto-renews, never expires
- **1Password fields:** app_key, app_secret, refresh_token
- **Fallback:** `$DROPBOX_OAUTH2_TOKEN` short-lived token (expires ~4h, avoid)



## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

Add whatever helps you do your job. This is your cheat sheet.

<!-- OPENCLAW_CACHE_BOUNDARY -->
