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

## Hindsight Memory Outage & Recovery (August 2, 2026)

**Problem:** No new memories captured from late May through August 1, 2026.

**Root Cause:** Hindsight service was timing out on all retain/recall operations:
- Service health check passed, but responses took >10 seconds
- Every retain call timed out at 10 seconds, silently failing
- LLM configuration was missing on the Hindsight service (only configured via environment, not persisted)
- Without an LLM, the service couldn't extract facts, so all operations blocked indefinitely

**Timeline:**
- **Late May 2026:** Hindsight stopped working (LLM config missing from service)
- **August 2, 2026 ~07:30 UTC:** Diagnosed timeouts and missing LLM config
- **August 2, 2026 ~07:35 UTC:** LLM configuration became active via environment variables; retention resumed
- **August 2, 2026 ~07:52 UTC:** Increased `recallTimeoutMs` from 10000 to 30000
- **August 2, 2026 ~07:53 UTC:** Recall started working — logs show "injecting 10 memories into context"

**Fix Applied:**
- Recall timeout increased to 30 seconds (gives semantic search enough headroom for the memory bank)
- Retention continues working as designed
- Auto-recall now successfully injects memories before agent responses
- Backfilled July 25 - Aug 1 sessions into Hindsight (424 sessions, 65 processed, 156+ pending)

**Status:** Fully operational. Memory gathering working, but recall latency is 12-15 seconds. ~2.5 months of data (May 31 - August 1) was lost during the outage, but all new conversations are being captured and recalled as of August 2.

**Verification:** Recent logs show consistent "injecting 5 memories" messages. Health check returns healthy. Agent knowledge tools show fresh memories being stored and retrieved from current session.

**Recall Performance Investigation (August 2, 01:14–01:27 UTC):**
- **Observation:** Recall operations consistently taking 12-15 seconds (CPU-bound in Hindsight process)
- **Root cause:** Hindsight queries all 1,085 memory units (across all fact_types) without pre-filtering. Forces PostgreSQL to compute vector distance for every row.
- **Index analysis:** HNSW indexes exist but cannot accelerate this query pattern because pgvector indexes work best with pre-filtered row sets. Hindsight's query lacks fact_type filtering.
- **Evidence:** Query with fact_type='observation' (83 rows): 2.3ms. Query across all types (1,085 rows): 14.8ms. Linear scaling with row count.
- **Potential fix:** If Hindsight filtered by fact_type during recall, would narrow to 80-500 rows and drop latency to <1s. Not implemented; would require Hindsight design changes.
- **Decision:** Accept 12-15s latency. Within 30s timeout, memories flowing, system stable. Further optimization blocked on Hindsight's recall query design.

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

## Browser Setup (Jul 25, 2026)

**Working config:** Local headless Chrome on `127.0.0.1:9222`
- Config file: `/home/node/.openclaw/openclaw.json` has `"browser": {"enabled": true, "cdpUrl": "http://127.0.0.1:9222"}`
- Startup: `/home/node/.cache/ms-playwright/chromium-1217/chrome-linux64/chrome --headless=new --remote-debugging-port=9222 --no-sandbox --disable-gpu --disable-dev-shm-usage`
- Verify with: `curl http://127.0.0.1:9222/json/version`
- If browser tool times out: check if Chrome process exists, restart with command above
- **Key lesson:** Config changes to `openclaw.json` require full process restart to reload (not just restart signals)

## Model Picker / Catalog (Oct 4, 2026)

- **`models.mode: "replace"` with empty `models.providers` = empty catalog** → model picker shows nothing (`openclaw models list` says "No models found") even though `/model claude-cli/...` still works. Fix: `models.mode: "merge"` (hot-reloads).
- The picker shows catalog rows filtered by `modelPolicy.allow`; `agents.defaults.models` keys must match catalog IDs exactly (e.g. `claude-cli/claude-haiku-4-5`, not the dated `-20251001` ID).
- `openclaw models list` reads the gateway's catalog — `OPENCLAW_CONFIG_PATH` scratch configs don't affect it, so test against the live config.
- Claude CLI models confirmed working: opus-5-5, opus-5, opus-4-8, opus-4-7, opus-4-6, sonnet-5-5, sonnet-5, sonnet-4-6, haiku-4-5. Not usable: fable-5/5-1 (needs usage credits), mythos-5.

## Hindsight Recall Latency Fix (Oct 4, 2026)

- **Cause of slow turns:** auto-recall blocked `before_prompt_build` for 11–14s every turn, before the claude-cli process even spawned. Trace showed the bank's **cross-encoder reranker** = 3.7–8.7s of a ~4–9s recall (retrieval itself ~0.1s). CPU-bound, varies with load.
- **Fix:** `PATCH http://hindsight:8888/v1/default/banks/dj/config` with `{"updates":{"enable_reranking":false}}` → recall ~0.3s. Relevance did not visibly degrade (reranked results were equally off-topic).
- **Revert:** same PATCH with `"enable_reranking": true`.
- **Separate issue (unfixed):** bank has 75 observations, last consolidation 2026-05-28, `pending_consolidation` 1225, 18 failed. Plugin recalls `observation` type by default, so recent (post-May) memories never surface in auto-recall. Fixing consolidation is the real relevance fix.
- **Hindsight infra (audit Oct 4, 2026):** v0.10.2, deployment `hindsight` in ns `openclaw` on the **microk8s** cluster. Embedded Postgres (pg0) on 1Gi hostpath PVC `hindsight-data` (~318MB), local CPU embeddings (bge-small) + reranker (MiniLM, now off), LLM = **`claude-code` provider, model `claude-haiku-4-5`** (switched Oct 4 2026 after Kiro was removed; auth = `CLAUDE_CODE_OAUTH_TOKEN` from `claude setup-token`, in secret `hindsight-secrets`; model name must be undotted `claude-haiku-4-5`, the Kiro-style `claude-haiku-4.5` fails). Image is `:latest` + pullPolicy Always, no resource limits, no probes. API :8888 unauthenticated in-cluster; control plane :9999 via ingress `hs.oc.ctb.padz.net` (basic auth).
- **DANGER: default `kubectl` context is `k8s` = Heru AKS (2020now/prod/stg), NOT where OpenClaw/Hindsight run.** Always pass `--context microk8s -n openclaw` explicitly; never switch the default.
- **retain_mission set on bank `dj` (Oct 4, 2026):** text in `projects/hindsight/retain_mission.txt`. Keeps durable facts (preferences/rules, decisions+why, technical root causes/gotchas, project status, personal context); skips assistant play-by-play, transient ops state/IDs, greetings, memory-system meta-talk, individual creative drafts. A/B tested on scratch banks: control kept noise (cron IDs, "Gizmo searched…"), mission kept only durable facts. Applies to FUTURE retains only; existing ~1,400 memories untouched. Revert: PATCH `/v1/default/banks/dj/config` `{"updates":{"retain_mission":null}}`. Another bank `sherra` exists (untouched).
- **Retain is slow via claude-code provider:** one extraction took ~2 min (each LLM call spawns a `claude` process). Fine for background retain; matters for consolidation of 1,225 pending memories.
- **Consolidation (fixed Oct 4 2026):** was stuck since 2026-05-28 by an orphaned `processing` op owned by a dead pod's random worker id. Fix = `DELETE /v1/default/banks/dj/operations/{id}` (documented to clear ops stranded by a crashed worker); worker id is now fixed (`HINDSIGHT_API_WORKER_ID=hs1`) so a restart reclaims its own tasks. Then it resumed: ~3.4 memories/min via claude-code Haiku (each batch of 5–8 = 60–160s LLM, sequential), ~6h for the 1,225 backlog; check `/banks/dj/stats` → `pending_consolidation`. **Complete Oct 4 2026 (~6:33 AM PT):** pending_consolidation=0, failed_consolidation=0 (after /consolidation/recover; it reported 0 retried but the earlier 18 cleared on the next run), total_observations=402 (was 75). Rounds of 100 chain on their own; only POST `/banks/dj/consolidate` if pending stops falling with no `processing` op.
- **Consolidation watcher (Oct 4 2026):** automation `hindsight-consolidation-watch` (id 95d2e4a0-6050-4a0c-994c-193213da6d00): condition trigger every 20 min (headless curl of bank dj stats, no model cost), once=true. Wakes an isolated agent turn on DONE / STALLED (no active op for 3 checks) / UNREACHABLE (6 failed checks); DONE branch = `/consolidation/recover` + `/consolidate`, recall spot-check, MEMORY.md update, then a Telegram message to Dj. Consolidation auto-continues in rounds of 100 (confirmed). Condition script was tested offline against 8 mocked scenarios. ETA ~13:00 UTC Oct 4.
- **Automation `kiro-model-monitor-daily` (id b7f53654-…) DISABLED Oct 4 2026** at Dj's request (Kiro removed; its payload model `kiro/claude-haiku-4.5` no longer exists). Not deleted; delete or re-point it later if wanted. `projects/kiro-model-monitor` is obsolete.
- **Nightly backup (rebuilt Oct 4 2026, LOCAL ONLY, 7 sets kept):** CronJob `openclaw-backup` (ns openclaw, microk8s ctx, 03:00 cluster time = 10:00 UTC) writes `/backups/<UTC ts>/{hindsight.dump,openclaw-dj.tar.gz,MANIFEST.sha256}` to PVC `openclaw-backups` (class `microk8s-hostpath-openclaw`, Retain; host path `/opt/openclaw/openclaw-openclaw-backups-pvc-14717434-…` on node `oc`). ~86MB/night. Keeps newest 7 complete sets; failed runs never evict good ones. **Same node/disk as everything else — not disaster recovery.** Live SQLite DBs are snapshotted with the sqlite backup API (not raw-tarred). Restore TESTED: Hindsight dump → scratch DB had identical row counts; tar hash/extract/sqlite integrity_check all OK. Docs: `projects/hindsight/BACKUP.md`; applied script `projects/hindsight/openclaw-backup.full.sh`. Azure was dropped (the old setup never worked: placeholder creds `AZURE_STORAGE_ACCOUNT=REPLACE_ME`, Azure-Linux/non-root image broke the kubectl install, SA can't `get deployments` so `exec deploy/x` is forbidden, so pods are resolved by label). Leftovers, safe to delete: secret `azure-backup-creds`, `projects/hindsight/*azure*` files. Sherra's instance is scaled to 0 so it is skipped. Excluded as regenerable: logs, cache, node_modules, venvs, .next, agents.old, media, tmp, and the two email-automation checkouts (Go binaries; source is in git). Unprotected-by-git loose ends noted Oct 4: apple2-emulator has no remote; toyota/trips 1 unpushed commit each; workspace/email-automation 1 modified file.
- Debug aids: `trace:true` on the recall request gives per-phase timings; plugin logs `perf: before_prompt_build hook_total=...` in openclaw-debug.log.

## Kiro Model Monitor (Jul 25, 2026)

Monitors https://kiro.dev/docs/models/ and keeps openclaw.json model catalog in sync, with a human approval gate.
- **Location:** `workspace/projects/kiro-model-monitor/` (stdlib-only Python, uv venv)
- **Drive it:** `./run.sh notify` (check + report), `./run.sh status`, `./run.sh apply` (writes openclaw.json + commits/pushes to main)
- **Approval flow:** daily cron 9am Pacific runs `notify` → if changes, messages Dj via Telegram → Dj says "approve models" → I run `./run.sh apply`
- **On demand:** Dj can ask "check for Kiro model updates" anytime
- **Kiro is the ONLY model source.** Anything not available through Kiro is superfluous. The `kiro-gateway /v1/models` endpoint (auth: KIRO_GATEWAY_TOKEN) is the authoritative source of *usable* models; the docs page is just human-readable metadata + "coming soon" heads-up. The monitor's PRIMARY signal is gateway-vs-config gap, not docs changes.
- **Key design:** costs NOT scraped (docs only show relative multiplier). Real USD costs come from `workspace/models-pricing.json` (hand-maintained, git-tracked); genuinely-new models get a placeholder cost flagged "COST REVIEW NEEDED". Auto (router) excluded from auto-add. Only gateway-live models auto-added; announced-but-not-live (e.g. GPT-5.6 tiers, Opus 5) held back.
- **ID normalization:** gateway `claude-sonnet-4` == config `claude-sonnet-4.0` (trailing `.0` stripped for equivalence) to avoid false gaps. Config allowlist may legitimately exceed the gateway's advertised list (e.g. claude-opus-4.8, claude-sonnet-5 are usable but not advertised) — shown as FYI only, never nagged.
- Cron job id: b7f53654-1330-43ab-ab3e-84d1ced55a9d

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

## Hindsight Auto-Retain Broken Since Sep 19 (diagnosed Oct 4, 2026)

- **Symptom:** last doc write to bank `dj` = 2026-09-19 17:53 UTC; recall/consolidation/health all fine, so it looked healthy. No log line at all on retain (silent).
- **Root cause = upstream plugin bug**, not our config/server: vectorize-io/hindsight issues #4537, #4721, #4828 (open; plugin 0.13.0 `latest` still has it). OpenClaw re-evaluates the plugin module per registry load (each cron run etc.), but `service.start()` runs once at gateway boot. Newest instance's `agent_end` -> `runRetain()` -> `retainLifecycleIsCurrent()` needs module-level `serviceAbortController` non-null (set only in `service.start`, dist/index.js ~L1709) -> returns silently. Recall survives via a lazy-init fallback (log: "waitForReady called before service.start()").
- **Why it began Sep 19-20:** plugin upgraded 0.10.0 -> 0.12.0 (0.10.0 has no lifecycle guard). Installed dir: `~/.openclaw/npm/projects/vectorize-io-hindsight-openclaw-c23cf52a67__openclaw-generation__g-9841c4ce4188339e/`.
- **Fix options:** (a) local patch of the guard at dist/index.js L2238 so an unstarted instance (controller null && serviceGeneration===0) counts as current; lost on plugin reinstall/upgrade; (b) pin back to 0.10.0 (verify compat with API 0.10.2 + `hooks.allowConversationAccess`); (c) wait for upstream. After fixing: backfill Sep 19 -> now with `hindsight-openclaw-backfill --dry-run` first (supports --resume/--checkpoint).
- Excluded providers `cron`,`dashboard` are intentional (not part of the bug).
- **FIX APPLIED + CONFIRMED (Oct 4 2026, 16:36Z):** local patch of `retainLifecycleIsCurrent` in plugin dist/index.js (unstarted instance, ctrl null && gen 0 => current; backup `index.js.orig-0.12.0`; also added `[LOCAL PATCH]` diagnostic logs). First retain after fix: log `Retained 2 messages to bank dj for session openclaw:agent:main:telegram:direct:8623402151`, bank docs 451->452. NOTE: the first post-reload turn (16:32Z) still dropped; it worked from 16:36Z. Plugin reloads itself on each agent run, so file edits apply without `plugins reload`. Patch is lost on plugin reinstall/upgrade: after any upgrade, check `grep -c 'LOCAL PATCH' dist/index.js` and re-verify retains (bank `dj` documents updated_at moves).
- **Backfill (Oct 4):** bundled `hindsight-openclaw-backfill` is JSONL-only and finds 0 sessions (OpenClaw now stores transcripts in SQLite `transcript_events`). Transcripts for Sep 19-Oct 4 05:00Z live in `~/.openclaw/agents.old/main/agent/openclaw-agent.sqlite` (live DB only has Oct 4 05:06Z on). agents.old `.jsonl.*`/`.zst` leftovers are pre-Sep-17 imports or cron noise (checked). Custom script `projects/hindsight/backfill-sqlite.ts` (gitignored dir): `node backfill-sqlite.ts` = dry-run, `--apply` enqueues; checkpoint `~/.openclaw/data/hindsight-sqlite-backfill-checkpoint.json` (resumable); 26 chunks / ~400k chars; ~3-4 min per chunk (claude-code Haiku extraction), so ~1.5h total.

## Lesson: don't call `plugins reload` mid-session (Oct 4 2026)
- My `plugins(action=reload, hindsight-openclaw)` at 16:31Z also reloaded other plugins (canvas etc.: log "plugin metadata changed with identical config; applying plugin lifecycle"). The session's `openclaw` MCP bridge then held retired plugin instances; from 16:35:29Z every request failed with HTTP 500 / JSON-RPC -32603 "Internal error", gateway log `request handling failed: Plugin canvas was reloaded or disabled; use its current tools.` (PluginInstanceUnavailableError, /app/dist/plugin-setup-module-*.mjs ~L862: "A retired instance cannot admit a fresh invocation"). All plugins/automations/message/etc. tools died, and new turns kept failing to connect. The reload tool even warned: "Start a new conversation to load changed tool definitions".
- Plugin dist edits are picked up on the next agent run anyway (plugin re-loads per run), so NO reload is needed after editing the plugin file. Recovery = new session or gateway restart (needs Dj's OK).
- `agents.old` (107MB) is NOT in the nightly backup (excluded) and holds the only copy of raw transcripts Sep 8 - Oct 4 05:00Z (live DB starts Oct 4 05:06Z). Don't delete before the Hindsight backfill finishes; archive (tar.zst) rather than delete.

## Gateway restart = exit, k8s respawns (Oct 4 2026)
- Dj asked that a gateway restart just clean up and exit (k8s respawns). Cause of in-process restarts: in a detected container with no supervisor markers, `restartGatewayProcessWithFreshPid` returns "disabled" ("container: use in-process restart to keep PID 1 alive"). Fix = env `OPENCLAW_SUPERVISOR_MODE=external` (detectGatewayRespawnSupervisor returns "external"): restart writes a SQLite handoff, flushes logs, exits 0; falls back to in-process if the handoff can't be persisted. Side effects: native service install/start/stop and self-update are refused (fine on k8s).
- Applied via `kubectl --context microk8s -n openclaw set env deploy/openclaw-dj OPENCLAW_SUPERVISOR_MODE=external` (deploy strategy Recreate; PID 1 is `sh -c docker-entrypoint.sh node openclaw.mjs gateway`, gateway is child PID 7). Not in any git manifest, so it lives only in the live Deployment. Revert: `kubectl ... set env deploy/openclaw-dj OPENCLAW_SUPERVISOR_MODE-`.
- VERIFIED Oct 4 2026 17:07Z: `/restart` logged "restart mode: full process restart (supervisor restart)", container exited 0 (Completed), k8s respawned it in the SAME pod/ReplicaSet (restartCount 0->1), gateway healthy again in ~1 min. External supervisor mode works as intended. Rollout of the env var was done earlier (~17:03Z).

## Doctor initContainer disabling skills (fixed Oct 4 2026)
- Symptom: every pod rollout, `openclaw-doctor` initContainer (`openclaw doctor --non-interactive --fix`) wrote `{"enabled": false}` for 9 skills (1password, blogwatcher, gh-issues, gifgrep, github, goplaces, session-logs, summarize, tmux) into openclaw.json. These are SKILLS not plugins.
- Cause: skill binaries live under /home/linuxbrew/.linuxbrew (and `~/.local/bin/homebrew-shim` scripts exec them); the gateway container mounts the `linuxbrew` volume but the doctor initContainer did not, and its PATH lacked it, so doctor judged them "missing binaries". From the gateway, `openclaw skills check` shows 0 missing.
- Fix applied: deployment `openclaw-dj` initContainers[1] now mounts `linuxbrew` at /home/linuxbrew/.linuxbrew/ and sets PATH incl. /home/linuxbrew/.linuxbrew/bin; the nine disabled entries were removed from openclaw.json beforehand (backup `openclaw.json.bak-*-pre-skillfix`). Rollback copy of the deployment: `projects/hindsight/openclaw-dj.deploy.pre-doctor-brewfix.yaml`. Untested whether doctor also needs the `syslib` mount; fallback = drop `--fix`. Verify after rollout: the nine skills should not reappear as `enabled:false`.

- **Telegram legacy webhook retired (Oct 4 2026):** ingress `/telegram-webhook` rule removed, `legacyWebhook=false`; details/rollback in `projects/openclaw-deploy/README.md`. Sherra's ingress untouched.

- **Hindsight retain re-check (Oct 5 2026, 00:21Z):** after the 00:13Z supervisor-mode restart, UNPATCHED plugin 0.13.0 retained the test turn (doc retained_at 00:17:56Z, log "Retained 2 messages to bank dj"). No LOCAL PATCH markers; patch not needed post-restart. The earlier failure was likely due to the in-process restart / load ordering (`service.start` not run in the newest instance). Still verify retains after any gateway restart or plugin upgrade (bank `dj` doc updated_at should move).
