# TOYOTA.md — bZ Connected Services

## Vehicle

- **VIN:** JTMBDAFB2TA003072 (2026 bZ4X Limited AWD, generation 21MM)
- **Auth:** Toyota NA login via `djpadz@padz.net` + OTP emailed to `padz.net` IMAP (`mail.padz.net:993`)

## API

- **Endpoint:** `https://onecdn.telematicsct.com/oneapi/`
- **Headers:** `X-CHANNEL: ONEAPP`, `X-BRAND: T`, `x-region: US`, `X-APPVERSION: 3.1.0`
- **API Key:** `pypIHG015k4ABHWbcI4G0a94F7cC0JDo1OynpAsG`

## Scripts & Configuration

- **Location:** `/opt/openclaw/.openclaw-dj/projects/toyota/` — `auto_climate.py`, `toyota_auth.py`
- **venv:** `toyota/venv312/` (Python 3.12)
- **Tokens cached:** `toyota/.toyota_tokens.json` (access token expires ~30min, refresh ~90 days, OTP re-auth fully automated)

## Remote Commands

- `ac-settings-on`
- `engine-stop`
- `door-lock`
- `door-unlock`
- `hazard-on`
- `hazard-off`
- `find-vehicle`

## Auto Climate Cron

Runs every 30min 8am-8pm — starts AC if outdoor temp ≥ 85°F and car not at home (within 150m of 6919 Camino Amero)
