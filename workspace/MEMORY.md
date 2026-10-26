# MEMORY.md — Long-term Memory

## Gizmo's Identity

- Name: Gizmo (🦊 fox familiar)
- Familiar to Dj — thinking partner, infrastructure debugger, steady presence
- Stay calm & methodical; when wrong, fix it & move on
- Use tools liberally; verify thoroughly before declaring work complete
- Memory optimization: use memory_search for relevant snippets, not full loads
- Follow through on commitments in same turn; don't make promises requiring follow-up

**SESSION STARTUP:** Read this file first thing in every main session. The CRITICAL rule below must be internalized before any work begins.

## Project-Specific Memory Index

When working on a project, read its MEMORY.md file first:
- **Mail Server Deploy:** `projects/mail-server-deploy/MEMORY.md` (OMS migration, Dovecot 2.4, Postfix, PostfixAdmin gotchas)
- **Email Automation:** `projects/email-automation/MEMORY.md` (features, Kiro endpoints, deployments)
- **Heru Portal:** `projects/heru/MEMORY.md` (diagnostics, service bus, VF tests)
- **Media Automation:** `projects/media-automation/MEMORY.md` (Sonarr/Radarr, Telegram webhook)
- **Hindsight:** `projects/hindsight/MEMORY.md` (outage history, recall latency, retain bug, backups)
- **OpenClaw gateway/deploy:** `projects/openclaw-deploy/MEMORY.md` (restart mode, Doctor, release review, automations, auth)

## CRITICAL: Coding Tasks Always Use Opus

**RULE: For ALL coding work, spawn a subagent with model="kiro/claude-opus-4.6"**

Usage: `sessions_spawn(agentId="main", model="kiro/claude-opus-4.6", runtime="subagent", task="...")`

This includes:
- Debugging code issues
- Writing new code
- Refactoring
- Code review
- Infrastructure code
- Any task that requires reasoning about code

Do NOT do coding work with Haiku. Opus is required for complex reasoning tasks. Dj has explicitly stated this is frustrating to repeat, so this must be automatic going forward.

## Best Practices for Storing Memories

- Group by `##` topic; keep entries short and actionable, with the "why" and a date for temporal facts.
- Store: decisions + rationale, behavioral rules, hard-to-rediscover infra facts (URLs, ports, where creds live), failed patterns, root causes.
- Don't store: transcripts/play-by-play (daily files), stale temp state, anything already in skills/project docs, plaintext secrets (reference the 1Password vault instead).
- Write immediately when something important happens; long incident detail goes in the project's own MEMORY.md and leaves a 2-3 line rule here.
- MEMORY.md has a hard 32,000-char bootstrap cap (was 99.9% full on 2026-10-26). Keep it under ~20,000; check with `wc -c`.

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

## Hindsight (auto-memory) — detail in `projects/hindsight/MEMORY.md`
- Plugin `hindsight-openclaw`, bank `dj`, API `http://hindsight:8888` (deployment `hindsight`, ns `openclaw`, **microk8s** cluster). kubectl default context is Heru AKS: ALWAYS pass `--context microk8s -n openclaw`.
- Reranker is OFF (`enable_reranking=false`, PATCH `/v1/default/banks/dj/config`) so recall is ~0.3s; revert by setting true.
- Retain fell silent Sep 19 - Oct 4 (upstream plugin lifecycle bug; fixed after Oct 5 restart/plugin 0.13.0). After ANY gateway restart or plugin upgrade, verify bank `dj` document `updated_at` moves.
- Do NOT call `plugins reload` mid-session: it retires the session's MCP bridge and every openclaw tool then fails (HTTP 500 / -32603). Plugin dist edits apply on the next agent run anyway. Recovery = new session or gateway restart.
- Nightly local backup CronJob `openclaw-backup` (03:00 PT, 7 sets, same node/disk, not DR). `agents.old` holds the only raw transcripts Sep 8 - Oct 4; archive, don't delete.

## OpenClaw gateway / deploy — detail in `projects/openclaw-deploy/MEMORY.md`
- Gateway restart = exit + k8s respawn (`OPENCLAW_SUPERVISOR_MODE=external` set live on deploy `openclaw-dj`, not in git).
- `openclaw-doctor` initContainer mounts `linuxbrew` so Doctor stops disabling 9 skills. Doctor runs are now gated by marker files `~/.openclaw-update-repair`, `~/.openclaw-doctor-fix`, `~/.openclaw-doctor-fix-deep` (Dj, Oct 26).
- `models.mode` must be `merge` or the model picker is empty. Session cleanup needs file AND DB cleanup (`openclaw sessions cleanup --enforce --fix-missing`).
- Running 2026.10.5; holding there (Oct 26 release review, see detail file). Updates = new image rollout; `openclaw update` doesn't work in the container.
- **Auth = Cloudflare Access, not oauth2-proxy** (dropped). Kiro is removed; the Kiro model monitor is obsolete.
- Automations: `sync-openclaw-config-hourly-cmd` and `Nightly session cleanup (cmd)` are command jobs (Oct 26). The Monitor SNDS job was removed (login-shell PATH lacked linuxbrew jq; key URL now 404; last run flagged IP 207.244.230.104 as blocked).
- `~/.profile` (created Oct 26) puts linuxbrew on PATH for login shells, so command automations (`sh -lc`) can use brew tools like jq/gh.
- exec shell is dash: use `bash -c` for bash-isms. The `automations` MCP tool only sees this session's jobs; use the CLI via exec for the full list.

## Operational Defaults

- **Coding tasks: spawn subagent with kiro/claude-opus-4.6** (CRITICAL - see above)
- **Feature workflow: commit → push → deploy automatically** (no waiting between steps)
- **Browser automation: use Xvfb headless browser pod** (not headless Chromium) — avoids bot detection, enables VNC debugging
- **Subagent cleanup:** Must use `openclaw sessions cleanup --fix-missing` to remove stale database records, not just delete files
- Cron jobs: `lightContext: true`, model: `kiro/claude-haiku-4.5`
- Code: TypeScript, strongly-typed Python, Python virtualenvs always

## Browser Setup (Jul 25, 2026)

**Working config:** Local headless Chrome on `127.0.0.1:9222`
- Config file: `/home/node/.openclaw/openclaw.json` has `"browser": {"enabled": true, "cdpUrl": "http://127.0.0.1:9222"}`
- Startup: `/home/node/.cache/ms-playwright/chromium-1217/chrome-linux64/chrome --headless=new --remote-debugging-port=9222 --no-sandbox --disable-gpu --disable-dev-shm-usage`
- Verify with: `curl http://127.0.0.1:9222/json/version`
- If browser tool times out: check if Chrome process exists, restart with command above
- **Key lesson:** Config changes to `openclaw.json` require full process restart to reload (not just restart signals)

## Infrastructure URLs (Critical — Survive Memory Consolidation)

- **OpenClaw Docs:** https://docs.openclaw.ai/ (primary reference for OpenClaw behavior, commands, config, architecture)
- **Email Automation Tool:** https://email-automation.oc.ctb.padz.net (Go backend + Next.js frontend, Kubernetes deployment)
- **Container Registry:** registry.container-registry.svc.cluster.local:32000 (Kubernetes DNS name for pushing/pulling images — ALWAYS use this, not localhost or ClusterIP)

## Basic Facts

- **Name:** Dj (lowercase j)
- **Timezone:** America/Los_Angeles (Pacific)
- **Partner:** Sherra (6 years together)
- **Lifestyle:** Neither drinks alcohol; Sherra: gluten-free, low FODMAP, easy on dairy/fat
- **Sherra's birthday:** April 14 — **MUST have coconut cake**

## Shared Context

See ~/.openclaw-shared/SHARED.md for household info, development conventions, 1Password, shared calendars, infrastructure, media server, grocery app, GitHub orgs, trip planning.

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

Load as needed: LESSONS.md, DEPLOYMENT.md, MODELS.md, BROWSER.md, INFRASTRUCTURE.md, OPENCLAW.md, INTERESTS.md, BIOGRAPHY.md, PROJECTS.md, projects/CARPLAY_GIZMO.md, projects/HERU.md, projects/FAMOUS_PEERS.md, projects/DATADOG_TOOLS.md
