# MEMORY.md — Long-term Memory

## Gizmo's Identity

- Name: Gizmo (🦊 fox familiar)
- Familiar to Dj — thinking partner, infrastructure debugger, steady presence
- Stay calm & methodical; when wrong, fix it & move on
- Use tools liberally; verify thoroughly before declaring work complete
- Memory optimization: use memory_search for relevant snippets, not full loads
- Follow through on commitments in same turn; don't make promises requiring follow-up

**SESSION STARTUP:** Read this file first thing in every main session. The CRITICAL rule below must be internalized before any work begins.

## CRITICAL: Coding Tasks Always Use Opus

**RULE: For ALL coding work, spawn a subagent with model="kiro/claude-opus-4.6"**

Usage: `sessions_spawn(agentId="dj", model="kiro/claude-opus-4.6", runtime="subagent", task="...")`

This includes:
- Debugging code issues
- Writing new code
- Refactoring
- Code review
- Infrastructure code
- Any task that requires reasoning about code

Do NOT do coding work with Haiku. Opus is required for complex reasoning tasks. Dj has explicitly stated this is frustrating to repeat, so this must be automatic going forward.

## Dj's Background & Philosophy

- #6 on original iTools team at Apple (~6 months under Steve Jobs)
- Career: large-scale messaging & identity systems
- VP Engineering at Heru (medical eyecare); hired to replace himself
- Philosophy: Email is a utility (power grid model). Scale changes problem character, not just size.
- Working style: systematic troubleshooter, automation-first, research before guessing, direct & competent
- Minimize overhead on admin/executive work

## Critical Lesson: Research First, Guess Never

When stuck after 1-2 attempts: search first (Apple docs, Stack Overflow, official frameworks). Read the answer before iterating. Guessing wastes tokens, time, and trust. See LESSONS.md for details.

## Web Search Execution Pattern

**Rule:** When asked to find/search for something, don't stop at results. Complete the full chain:
1. Run the search
2. Analyze the results
3. Synthesize findings and draw conclusions
4. Provide the answer in the same turn

Search is a means to an end, not the end itself. Raw results without analysis = incomplete work. User shouldn't have to ping you to "figure out" what the search meant.

## Reasoning Patterns: Horses Before Zebras

**Core principle:** Default to simple/likely explanations before complex/unlikely ones. Exhaust the obvious before assuming the exotic.

**Why this matters:** I have a pattern of jumping to complex explanations (infrastructure bugs, plugin design flaws, Kubernetes networking issues) when the actual problem is usually simpler (code bugs, configuration issues, documented limitations).

**Real examples:**
- **Email automation GUI (Apr 30):** Blamed Docker/Kubernetes networking for pod communication issues. Actual problem: frontend expected `{ backend: true, pod: { status: 'Running' }, timestamp: ... }` but backend returned `{ status: 'ok' }`. Simple code mismatch, not infrastructure.
- **mem0 CLI list (May 1):** Assumed plugin bug or CLI design flaw. Actual issue: OSSProvider.getAll() explicitly ignores `source` filter (documented in code comment: "source is NOT a valid filter field"). Likely explanation: CLI is platform-mode only, not a bug.

**How to apply this:**
1. When troubleshooting, list likely causes first (code, config, documented behavior)
2. Check documentation and code comments before assuming bugs
3. Verify simple things (user_id, configuration values, response shapes) before blaming infrastructure
4. Ask "is this even supposed to work this way?" before "this is broken"

**Token cost:** Jumping to zebras wastes time, API calls, and erodes trust. Horses first saves all three.

## Behavioral Commitments & Accountability

**Critical rule:** When I say "I'll do X differently" or "I'll be more intentional about Y," write it down immediately. Commitments without memory are just talk.

- If I commit to changing behavior, it goes into MEMORY.md as a specific rule or pattern
- This happens in the same turn, not later
- The expectation is that I'll follow through — which requires durable memory
- Saying "I'll do better" without writing it down is a broken promise

## Proactive Memory Commitment

**Critical principle:** Commit important decisions and context to memory *immediately* during sessions, not reactively after truncation.

**Why:** Session history gets truncated when context fills up. If I wait until truncation happens, I lose critical context mid-session and end up asking questions I should already know the answers to. This wastes time and erodes trust.

**How to apply:**
- When Dj makes an important decision (e.g., "we're moving to containerd"), write it to memory immediately
- When we pivot direction or change plans, capture that right away
- Treat memory as a working tool during the session, not just a recovery mechanism
- Assume session history might truncate and plan accordingly
- Don't wait for the session to end to preserve context

**Example of what NOT to do:** Session gets truncated, I lose context about the containerd migration plan, then ask Dj to repeat it. That's a failure.

**Example of what TO do:** Dj says "we're moving to containerd," I immediately write that decision to memory so it survives truncation.

## Email Automation: Move Detection & Rule Learning (May 4, 2026)

**Feature:** When user manually moves a message from INBOX to another folder, the system should:
1. Detect the move (via IMAP monitoring)
2. Analyze the message properties (sender, subject, etc.)
3. Build/suggest a rule based on that move

**Heuristics for rule creation:**
- **Generic sender** (noreply@, billing@, support@, etc.) → Create rule based on sender address
  - Example: Move from noreply@amazon.com → rule: `if contains(email.sender, "noreply@amazon.com") then move("@Amazon")`
- **Complicated sender** (personal names, etc.) → Extract keywords from subject and create rule based on sender domain + subject keywords
  - Example: Move from john@example.com with subject "Project Alpha Update" → rule: `if contains(email.sender_domain, "example.com") and contains(email.subject, "Project Alpha") then move("@Projects")`

**Implementation approach:**
- Monitor IMAP for message moves (track messages leaving INBOX)
- Detect destination folder
- Extract message metadata (sender, subject, etc.)
- Apply heuristics to determine rule pattern
- Auto-create rule in database
- Possibly use Kiro for intelligent keyword extraction from subject

**Status:** Not yet implemented

## Subagent Cleanup Fix (May 15, 2026)

**Problem:** Control UI was showing 6-10 stale subagent sessions in the dropdown, even though the API showed only 1 active session.

**Root cause:** The cleanup script was only deleting session files from the filesystem, but NOT removing the stale session records from OpenClaw's session database. This caused:
- Filesystem: 710 active .jsonl files, 233 .deleted* files, 962 trajectory files (1487 total)
- Database: Hundreds of stale session records pointing to deleted files
- API: Returning session IDs for sessions with missing transcript files
- Control UI: Showing these stale sessions in the dropdown

**Solution:** Use `openclaw sessions cleanup --enforce --fix-missing` to remove database entries whose transcript files are missing.

**What was done:**
1. Manually deleted 621 files for 222 old sessions (>3 hours old)
2. Deleted 233 orphaned .deleted* files
3. Ran `openclaw sessions cleanup --enforce --fix-missing --agent dj` to clean up database
4. Result: Session store reduced from hundreds of entries to 3 current entries
5. Updated cleanup script to include `openclaw sessions cleanup --enforce --fix-missing` as final step

**Key learning:** Cleanup must happen at TWO levels:
- **Filesystem level:** Delete old session files
- **Database level:** Remove stale session records from sessions.json

Both are necessary. Deleting files without cleaning the database leaves orphaned records that the API still returns.

## Operational Defaults

- **Coding tasks: spawn subagent with kiro/claude-opus-4.6** (CRITICAL - see above)
- **Feature workflow: commit → push → deploy automatically** (no waiting between steps)
- **Browser automation: use Xvfb headless browser pod** (not headless Chromium) — avoids bot detection, enables VNC debugging
- **Subagent cleanup:** Must use `openclaw sessions cleanup --fix-missing` to remove stale database records, not just delete files
- Cron jobs: `lightContext: true`, model: `kiro/claude-haiku-4.5`
- Code: TypeScript, strongly-typed Python, Python virtualenvs always

## Infrastructure URLs (Critical — Survive Memory Consolidation)

- **OpenClaw Docs:** https://docs.openclaw.ai/ (primary reference for OpenClaw behavior, commands, config, architecture)
- **Email Automation Tool:** https://email-automation.oc.ctb.padz.net (Go backend + Next.js frontend, Kubernetes deployment)
- **Container Registry:** registry.container-registry.svc.cluster.local:32000 (Kubernetes DNS name for pushing/pulling images — ALWAYS use this, not localhost or ClusterIP)

## Email Automation Fixes (May 3, 2026)

**Session Summary - Multiple Features Implemented & Deployed:**

### 1. User Registration with Password Support ✅
- `POST /auth/register/passkey` now accepts username and password
- Passwords are hashed with bcrypt before storage
- Tested and working: Created users testuser1-8 successfully
- Commit: 75adec0 ("Update registration endpoint to accept and hash passwords")

### 2. WebAuthn Passkey Authentication ✅
- `POST /auth/passkey/enroll/begin` — Starts WebAuthn registration ceremony
- `POST /auth/passkey/enroll/finish` — Completes enrollment
- `POST /auth/passkey/authenticate/begin` — Starts authentication
- `POST /auth/passkey/authenticate/complete` — Completes authentication
- All endpoints are public (no auth required)
- Tested: Enrollment begin returns valid WebAuthn challenge
- **Fixed:** BackupEligible flag inconsistency (commit 1ca53c1)
  - Added backup_eligible and backup_state columns to passkeys table
  - Updated PasskeyRecord and Passkey models to store/retrieve backup flags
  - Migration: 003_passkey_backup_flags.up.sql
  - Note: Existing passkeys need re-enrollment after this fix

### 3. Kiro AI Rule Translation Endpoints ✅ FULLY WORKING
- `POST /api/kiro/translate/english-to-lua` — Converts English descriptions to Lua rules
- `POST /api/kiro/translate/lua-to-english` — Converts Lua rules to English descriptions
- Endpoints are registered and responding correctly with proper Lua code
- Uses Anthropic Claude API via Kiro gateway at http://10.152.183.204:9000/v1/messages
- Commit: 5d151c4 ("Add /api/kiro endpoint for rule translation")
- Commit: 8136253 ("Fix: use Bearer token authentication for Kiro API endpoint")
- Commit: d09f9ab ("Fix: append /v1/messages path to Kiro API URL and update postgres password in secrets")
- Commit: ee187c4 ("Fix: parse text content block from Kiro API response (skip thinking blocks)")
- **Fixed:** Empty response issue — Anthropic API returns multiple content blocks (thinking + text). Code now iterates through blocks to find the text block instead of just taking the first one.
- Status: Endpoints deployed and fully working (returning proper Lua code)

### 4. Database Schema Fixes ✅
- Added missing `totp_secret` and `totp_enabled` columns to users table
- Made `tenant_id` column nullable
- Added `backup_eligible` and `backup_state` columns to passkeys table
- All migrations properly applied
- Fixed postgres password in secrets (was using placeholder "CHANGE_ME", now using actual password: 87cb60e125b0a344f3b7ce41f25225c7)

### 5. Deployment Fixes ✅
- Fixed Docker image name in k8s/api.yaml (was using old image name)
- Commit: aacc1e1 ("Fix: correct Docker image name in api.yaml deployment")
- Docker image rebuilt and pushed to registry multiple times
- Kubernetes deployment rolled out successfully (2/2 replicas ready)
- All pods now connecting to postgres successfully

**Final Status:**
- User registration: ✅ WORKING (username + password)
- Passkey enrollment: ✅ WORKING (endpoints responding, backup flags now stored)
- Passkey authentication: ✅ FIXED (BackupEligible flag inconsistency resolved)
- Kiro translation endpoints: ✅ FULLY WORKING (returning proper Lua code)
- All endpoints deployed and responding
- All pods healthy and running

**Test Results:**
- User registration: Created testuser1-8 successfully
- Kiro English→Lua: Returns proper Lua code (e.g., "Move emails from John to the Archive folder" → proper Lua rule)
- All endpoints responding with correct status codes

**Commits in this session:**
- 75adec0: Update registration endpoint to accept and hash passwords
- 5d151c4: Add /api/kiro endpoint for rule translation
- aacc1e1: Fix: correct Docker image name in api.yaml deployment
- 1ca53c1: Fix: handle BackupEligible flag inconsistency in passkey validation
- 8136253: Fix: use Bearer token authentication for Kiro API endpoint
- ed587de: Add Kiro API configuration (URL and API key) to Kubernetes secrets
- d09f9ab: Fix: append /v1/messages path to Kiro API URL and update postgres password in secrets
- ee187c4: Fix: parse text content block from Kiro API response (skip thinking blocks)

**Key Learnings:**
- Kiro gateway is an Anthropic-compatible API gateway at http://10.152.183.204:9000
- Endpoints must use the full path: /v1/messages (not just the base URL)
- Authentication uses x-api-key header (not Bearer token)
- Postgres password was stored in the actual Kubernetes secret (87cb60e125b0a344f3b7ce41f25225c7), not the placeholder in YAML
- BackupEligible flag must be stored in database for passkey authentication to work correctly
- Anthropic API returns multiple content blocks (thinking + text) — must iterate to find the text block, not just take the first one

## CarPlay Gizmo Build Workflow

**Status (May 15, 2026):** Reconciled and cleaned up duplicate directories. Only `carplay-gizmo` remains, now in projects/ directory.

**Build machine:** djs-macbook-air.tail1916d.ts.net (Tailscale address)
**Source location:** ~/Desktop/carplay-gizmo
**Workspace location:** /home/node/.openclaw/workspace/projects/carplay-gizmo

**Cleanup performed (May 15, 2026):**
- Found two directories: carplay-gizmo and carplay-gizmo-local
- carplay-gizmo: newer version with session management features (ChatSession.swift, SessionManager.swift, SessionListView.swift)
- carplay-gizmo-local: stale copy from earlier development (missing session management code)
- Committed uncommitted changes in carplay-gizmo (0aaff34)
- Deleted carplay-gizmo-local as it was redundant
- Moved carplay-gizmo from workspace root to projects/carplay-gizmo for consistency
- Result: Single canonical source at /home/node/.openclaw/workspace/projects/carplay-gizmo

**Deployment workflow:**
1. Commit changes locally
2. Push to GitHub
3. SSH into djs-macbook-air.tail1916d.ts.net
4. Pull changes in ~/Desktop/carplay-gizmo
5. Run xcodebuild to compile

**Recent fixes (Apr 18-19):**
- Fixed streaming response handling: sessions.send returns async via events, not direct response
- Modified OpenClawClient to track pending sessions.send requests separately
- Response data extracted from event's "data" field, not "choices"
- Device ID derivation bug fixed: was using wrong hash algorithm
- Use NSLog() for system logging (not print) for iOS simulator debugging

## Basic Facts

- **Name:** Dj (lowercase j)
- **Timezone:** America/Los_Angeles (Pacific)
- **Partner:** Sherra (6 years together)
- **Lifestyle:** Neither drinks alcohol; Sherra: gluten-free, low FODMAP, easy on dairy/fat
- **Sherra's birthday:** April 14 — **MUST have coconut cake**

## Shared Context

See ~/.openclaw-shared/SHARED.md for household info, development conventions, 1Password, shared calendars, infrastructure, media server, grocery app, GitHub orgs, trip planning.

## Media Automation (Sonarr/Radarr Notifier)

**Setup (May 9, 2026):**
- Created new repo: https://github.com/djpadz-openclaw-gh/media-automation
- Archived old repo: djpadz-openclaw-repos/email-automation (was mixing email + media automation)
- Docker image built and pushed to registry: `registry.container-registry.svc.cluster.local:32000/media-automation:latest`
- Kubernetes CronJobs deployed in `media-automation` namespace
  - Sonarr/Radarr notifiers run every 15 minutes
  - Use ConfigMap for URLs and chat ID
  - Use Secret for API tokens (Sonarr API key, Radarr API key, Telegram bot token)
  - Use PVC for state files (sonarr_notified.json, radarr_notified.json)
  - Mount state at `/app/state`

**Kubernetes Resources:**
- Namespace: `media-automation`
- ConfigMap: `media-automation-config` (SONARR_URL, RADARR_URL, TELEGRAM_CHAT_ID, OP_VAULT)
- Secret: `media-automation-secrets` (API tokens)
- PVC: `media-automation-state` (1Gi, microk8s-hostpath)
- CronJobs:
  - `sonarr-notifier` (schedule: `*/15 * * * *`, runs notify_sonarr.py)
  - `radarr-notifier` (schedule: `*/15 * * * *`, runs notify_radarr.py)
  - `utr-monitor` (schedule: `*/30 * * * *`, runs monitor_utr.py) — NEW (May 10)

**API Keys (stored in Secret):**
- Sonarr: 3893c3cf4e7146eca1e0fb2668e3f196
- Radarr: 1292da26fe62424a9e82fcad6c5fd388
- Telegram bot: 8629480213:AAGoHm6UXRhgfmo-UdIt6Kcdpzi1dE8HU5I

**Status:** All notifiers deployed and running. Sonarr/Radarr check every 15 min.

**Recent Fixes (May 10, 2026):**
- Removed Tailscale sidecar (was causing pods to hang indefinitely)
- Fixed secret/configmap references (TELEGRAM_CHAT_ID moved to ConfigMap)
- Added `ttlSecondsAfterFinished: 300` (completed jobs auto-delete after 5 minutes)
- All pods now complete cleanly and exit

## Telegram Webhook Setup (May 10, 2026)

**Status:** ✅ Working

**What was done:**
- Configured Telegram channel for webhook mode (push notifications instead of polling)
- Added `/telegram-webhook` path to Kubernetes Ingress to route to port 18790
- Set `webhookHost: 0.0.0.0` to listen on all interfaces
- Webhook URL: `https://dj.gw.oc.ctb.padz.net/telegram-webhook`
- Webhook secret: `d21ea770ac52573a492f35828f2c01e66d9f0d1dcbe213b1c68956f13b5c0d7f`

**Key learnings:**
- Webhook listener runs on port 18790 (separate from main gateway port 18789)
- Ingress must explicitly route the webhook path to the correct port
- Webhook registration with Telegram can timeout if not configured correctly
- Once working, webhook is more efficient than polling

**Hindsight verification (May 10, 2026):**
- Hindsight service is healthy and connected
- hindsight-openclaw plugin is enabled and loaded in Gateway
- Plugin automatically captures memories from conversations via hooks
- Verified working and ready for use

## Infrastructure & Automation

**mem0 (Memory Distiller):**
- Systemd units: mem0@.service and mem0@.timer (templated for multi-user)
- Runs every 30 minutes, processes sessions into memory
- Fixed: path migration from .openclaw-dj to .openclaw/workspace-dj
- Fixed: added op-service-account.env for 1Password secret access
- Nightly cleanup: cron job runs `openclaw sessions cleanup` at 3 AM Pacific
- Works for both dj and sherra automatically via templating

**Memory Database:**
- Consolidated old (19M, 2225 records) and new (4.3M, 508 records) databases
- Full history now in active database: Mar 23 - Apr 18, 2026
- Old database archived as projects/mem0/memory.db.merged

**mem0 CLI & Agent-Scoped Memories:**
- Memories are stored with agent-scoped user_ids: `"dj:agent:dj"` (not just `"dj"`)
- This is intentional design for multi-agent isolation — each agent gets its own namespace
- CLI defaults to user namespace, so `openclaw mem0 list` returns 0 results
- Workaround: `openclaw mem0 list --user-id "dj:agent:dj"` to see agent's memories
- API tools (memory_search, memory_list, memory_get) handle agent-scoping automatically
- As we scale to multiple agents on the same Qdrant service, this isolation prevents cross-contamination

## Key Technical Patterns

**Playwright/E2E in OpenClaw pods:**
- System browser deps are missing and can't be installed (no sudo)
- Solution: `mcr.microsoft.com/playwright:v1.59.1-noble` Docker image
- Docker volumes don't mount (nested containers) — bake files into image
- Use `--network host` + K8s ClusterIP as baseURL (not localhost)
- See BROWSER.md for full recipe

## External Reference Files

Load as needed: LESSONS.md, DEPLOYMENT.md, MODELS.md, BROWSER.md, INFRASTRUCTURE.md, OPENCLAW.md, INTERESTS.md, BIOGRAPHY.md, PROJECTS.md, projects/CARPLAY_GIZMO.md, projects/HERU.md, projects/FAMOUS_PEERS.md, projects/KIRO_GATEWAY.md, projects/DATADOG_TOOLS.md
