# Heartbeat Checks

## Weather

Check current weather and forecast for San Diego. Relevant for:
- Whether Dj might go out
- Trip planning
- Outdoor activity feasibility

Use the weather skill: search for "San Diego weather" or just fetch current conditions.

---

**Rotation:** Check weather every other heartbeat. Last checked: [will update during runs]

---

## TEMPORARY: Hindsight backfill follow-up (added Oct 4 2026 ~9:45 AM PT; delete this section when done)

Dj approved a backfill of Sep 19 -> Oct 4 transcripts into Hindsight bank `dj`; it was started ~16:36Z Oct 4 and promised a report when finished. Every heartbeat until done:
1. Count chunks: `python3 -c "import json;print(len(json.load(open('/home/node/.openclaw/data/hindsight-sqlite-backfill-checkpoint.json'))))"` (target 26) and `pgrep -f backfill-sqlite`.
2. `curl -s http://hindsight:8888/v1/default/banks/dj/stats` -> pending_operations, failed_operations (baseline failed = 16 old ones; new failures = problem).
3. If the process died before 26/26: re-run `cd ~/.openclaw/workspace/projects/hindsight && nohup node --disable-warning=MODULE_TYPELESS_PACKAGE_JSON backfill-sqlite.ts --apply >> backfill-apply.log 2>&1 &` (resumable).
4. When 26/26 enqueued AND pending_operations == 0: spot-check recall (e.g. "Canvas on djs-16-mbp", "Apple Watch Series 12") returns post-Sep-19 items, confirm live retain still working (bank docs updated_at moving; log "Retained N messages"), then message Dj a short result via Telegram, and delete this section.
Also tell Dj if anything failed. Details: MEMORY.md "Hindsight Auto-Retain Broken Since Sep 19", memory/2026-10-04.md.
