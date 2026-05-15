# SCAMBAITER.md — Beverly Telegram Userbot

Telegram userbot running as your actual account via Telethon (not a separate bot). Deploys "Beverly" — a warm, confused, perpetually-distracted retiree persona who engages scammers in DMs to waste their time and gather intelligence.

## Why Userbot (Not a Bot Account)

- Bots are easier for Telegram to detect and ban
- Userbot on your real account is stealthier and harder to flag
- Scammers think they're talking to a real person, not automation
- Control stays private (via Saved Messages commands)

## Why Not Gizmo

- ScamBaiter is a separate tool with its own session, state management, and logging
- Mixing it with Gizmo would expose the automation and complicate both systems
- Beverly needs independent character consistency and conversation history
- Gizmo is your main assistant; ScamBaiter is a specialized tactical tool

## Control Mechanism (via Saved Messages)

- `/bait @username` — start Beverly on that scammer
- `/stopbait @username` — stop the session
- `/baitlist` — list active sessions
- Beverly auto-stops after 3 hours of scammer silence

## Beverly's Character

- Warm, curious, slightly gullible but not obviously stupid
- Perpetually confused and distracted (tangents about Mr. Pickles the cat, garden, late husband Gerald, grandkids)
- Always needs to "check with my son first" before committing
- Short responses (1-3 sentences), occasional typos, no abbreviations
- Types like a real older person
- Varies reactions to avoid bot patterns
- Never commits or closes anything — always one more question

## Setup

- **Location:** `~/.openclaw-shared/scambaiter/`
- **GitHub:** https://github.com/djpadz-openclaw-repos/scambaiter
- **Credentials:** Telegram API ID/Hash in `.env` or 1Password (`Telegram API` item)
- **First run:** Interactive (phone auth via Telethon)
- **Subsequent runs:** `./start.sh` or systemd service
- **LLM:** Uses Kiro for responses (via Anthropic SDK with `base_url` + `default_headers`)
- **venv:** `.venv/` with telethon and anthropic installed

**Fixed 2026-04-10:**
- Local copy was not a git clone (had .gitignore but no .git directory)
- Cloned from GitHub to make it version-controlled
- Restored local state files (.env, session/, logs/, state.json, .venv)
- Now properly synced with GitHub repo

## REST API for Beverly Control

**Purpose:** Allow Gizmo to trigger bait sessions programmatically without exposing automation

**Architecture:** Stateless FastAPI server that reads/writes to shared `state.json`

**Deployment:** k8s namespace `scambaiter`, NodePort 30800 (internal only, firewall blocks external)

**Endpoints:**
- `POST /api/bait/start` — Start baiting a user (body: `{"username": "@user", "description": "optional"}`)
- `POST /api/bait/stop` — Stop baiting a user (body: `{"username": "@user"}`)
- `GET /api/bait/status` — List active sessions
- `GET /health` — Health check

**Client Library:** `scambaiter_client.py` in repo root
```python
from scambaiter_client import ScambaiterClient
client = ScambaiterClient("http://localhost:30800")
client.start_bait("@username", description="...")
client.stop_bait("@username")
client.get_status()
```

**How it works:**
1. Gizmo calls API to start/stop sessions
2. API updates `state.json` with session metadata
3. Main scambaiter.py process watches `state.json` and picks up changes
4. Beverly engages independently — scammers see no coordination pattern
5. API is stateless, so it can be restarted without losing sessions

**Security:** API only exposed internally via NodePort; firewall blocks external access

**Docker:** `Dockerfile` builds image, pushed to `localhost:32000/scambaiter-api:latest`

**Python 3.14:** Upgraded from 3.11 for future-proofing (2026-04-10)

## ScamBaiter Skill for Gizmo

**Location:** `~/.openclaw-shared/skills/scambaiter/`

**Purpose:** Provides `/bait` command interface for Gizmo to control Beverly

**Commands:**
- `/bait @username` — Start baiting a user
- `/bait stop @username` — Stop baiting a user
- `/bait status` — List active sessions

**Implementation:**
- `scambaiter_skill.py`: Command parser and API wrapper
- `SKILL.md`: Documentation
- `__init__.py`: Package initialization
- `references/example_usage.py`: Usage examples

**How Gizmo uses it:**
1. User sends Gizmo: `/bait @scammer_username`
2. Gizmo parses the command via the skill
3. Skill calls ScamBaiter API to start session
4. Beverly engages independently
5. User stays in control — no auto-detection, no false positives

**Design principle:** Explicit control only. No automatic scam detection (false positive cost too high)

## Multi-Account Support (2026-04-10) — COMPLETE

**Architecture:** One API instance (k8s), multiple Telethon clients (one per account)

**State:** Organized by account in `state.json` — completely independent sessions

**API v2.0:** All endpoints accept `?account=dj` query parameter (defaults to 'dj')
- `POST /api/bait/start?account=dj` — Start baiting
- `POST /api/bait/stop?account=dj` — Stop baiting
- `GET /api/bait/status?account=dj` — List active sessions
- `GET /health` — Health check

**Main Process:** `scambaiter_multi.py` manages multiple accounts
- Loads accounts from `SCAMBAITER_ACCOUNTS` environment variable
- Each account has separate Telethon client and session file
- `handle_command()` — Parse `/bait` commands from Saved Messages
- `handle_incoming_message()` — Engage scammers with Beverly using Claude API via Kiro
- Maintains conversation history per session
- Updates state after each interaction

**Client Library:** `scambaiter_client.py` updated to support account parameter
- `start_bait(username, account="dj")`
- `stop_bait(username, account="dj")`
- `get_status(account="dj")`

**Skill:** `scambaiter_skill.py` updated to support account parameter
- `/bait @username` — Start baiting
- `/bait stop @username` — Stop baiting
- `/bait status` — List active sessions

**Documentation:**
- `SETUP_MULTI_ACCOUNT.md` — Complete setup guide for new users
- `README_MULTI_ACCOUNT.md` — System overview, architecture, deployment
- `CONFIG.md` — Configuration reference

**Deployment:**
- Docker image rebuilt with multi-account support
- Redeployed to k8s (namespace: scambaiter, port: 30800)
- Verified independent state works for both accounts

**Production Ready:**
- Dj running on his account (already configured)
- Sherra can run on her account (setup guide provided)
- Both use shared API infrastructure
- Independent Beverly instances per user
- Full engagement logic implemented
- Tested and verified working
