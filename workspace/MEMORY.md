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
- **Email Automation:** `projects/email-automation/MEMORY.md` (features, Kiro endpoints, deployments)
- **Heru Portal:** `projects/heru/MEMORY.md` (diagnostics, service bus, VF tests)
- **Media Automation:** `projects/media-automation/MEMORY.md` (Sonarr/Radarr, Telegram webhook)

These files are loaded on-demand to keep main context lean.

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

## Best Practices for Storing Memories

**Structure:**
- Use clear headers (`##`) to group related memories by topic
- Keep entries concise and actionable — distill lessons, don't dump transcripts
- Include "why" alongside "what" so future-me understands the reasoning
- Date-stamp temporal entries (decisions, fixes, status changes)
- Reference external files for lengthy details rather than inlining everything

**What to store:**
- Decisions and their rationale
- Behavioral rules and commitments (with specific examples)
- Infrastructure facts that are hard to rediscover (URLs, credentials locations, port numbers)
- Patterns that failed and why (prevents repeat mistakes)
- Key technical learnings from debugging sessions
- Relationship/personal context that affects interactions

**What NOT to store:**
- Raw session transcripts or play-by-play logs (use daily memory files for that)
- Temporary state that will be stale in a week (unless date-stamped with expiry intent)
- Information already captured in skill files or project docs (just reference them)
- Secrets/credentials in plaintext (reference 1Password vault locations instead)

**Maintenance:**
- Prune stale entries during heartbeat memory reviews
- Consolidate related entries that have grown scattered
- Move completed project details to project-specific files when they get long
- Keep critical rules (like the Opus rule) near the top where they can't be missed
- When a section grows past ~20 lines, consider whether it belongs in its own file

**Timing:**
- Write immediately when something important happens — don't defer
- Commitments without memory are just talk (see Behavioral Commitments section)
- Assume context will truncate; capture decisions as they're made

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

Load as needed: LESSONS.md, DEPLOYMENT.md, MODELS.md, BROWSER.md, INFRASTRUCTURE.md, OPENCLAW.md, INTERESTS.md, BIOGRAPHY.md, PROJECTS.md, projects/CARPLAY_GIZMO.md, projects/HERU.md, projects/FAMOUS_PEERS.md, projects/KIRO_GATEWAY.md, projects/DATADOG_TOOLS.md
