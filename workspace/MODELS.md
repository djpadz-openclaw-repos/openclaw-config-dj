# MODELS.md — AI Provider & Model Selection

## AI Provider

**Kiro** is the AI provider — local proxy at `http://localhost:9000`, uses Bearer token auth (`Authorization: Bearer <token>`).

- Anthropic API key has been **removed from 1Password**. Do NOT call `api.anthropic.com` directly.
- Kiro API token stored in 1Password: item **"Kiro API"**, vault **OpenClaw**, field **credential**.
- When scripts call an LLM directly (Python/TypeScript), use Kiro endpoint + Bearer auth. The Anthropic SDK supports this via `base_url` + `default_headers`.
- Email automation scripts (`notify_actionable_emails.py`, `notify_teams_slack.py`, `actionable_email_daemon.py`) updated to use Kiro.
- Scambaiter (`scambaiter.py`) updated to use Kiro via `base_url` + `default_headers` on Anthropic SDK.
- Sherra's `extract.ts` (mem0) updated to use Kiro.
- Both `secrets.sh` files (dj + sherra) now fetch Kiro token from 1Password "Kiro API" item.

## Model Selection Rules

**Cost-Aware Model Selection** — All models route through Kiro (`http://localhost:9000`)

Cost multipliers (relative to baseline):
- Qwen3 Coder Next: **0.05x** (cheapest)
- MiniMax M2.1: 0.15x
- DeepSeek 3.2: 0.25x
- MiniMax M2.5: 0.25x
- Claude Haiku 4.5: 0.4x
- GLM-5: 0.5x
- Auto: 1.0x (baseline, routes to optimal model per task)
- Claude Sonnet 4.0/4.5/4.6: 1.3x
- Claude Opus 4.5/4.6: 2.2x (most expensive)

### Tier 1 — Coding: MiniMax M2.5 `kiro/minimax-m2.5` (0.25x)
- Creating new code files
- Modifying/refactoring existing code
- Debugging code issues
- Code reviews
- **Trial approach:** Use MiniMax M2.5 (5x cheaper than Sonnet). If it proves problematic, fallback to Sonnet 4.6 or Opus 4.6.
- MiniMax M2.5 is described as "Near Opus-level coding results at a fraction of the cost"

### Tier 2 — Real Work: Haiku 4.5 `kiro/claude-haiku-4.5` (0.4x)
- Cron jobs with real consequences (reservation booking, email classification)
- Tasks requiring reliable reasoning or edge case handling
- Anything where failure is costly
- Reading/exploring code (not modifying)
- General conversation and questions

### Tier 3 — Cron Jobs (Auto-routing): Auto `kiro/auto-kiro` (1.0x baseline)
- Health checks, heartbeats, stock checks, pattern matching
- Script execution jobs
- Auto routes to the optimal model per task, balancing quality and cost
- Cheaper on average than fixed models for simple tasks
- No predictability needed for these jobs

### Available Kiro models (only use these)
- `kiro/minimax-m2.5` (0.25x) — coding (trial)
- `kiro/claude-sonnet-4.6` (1.3x) — coding fallback
- `kiro/claude-opus-4.6` (2.2x) — coding fallback for very complex tasks
- `kiro/claude-haiku-4.5` (0.4x) — real work with consequences
- `kiro/auto-kiro` (1.0x) — cron jobs, auto-routing
- `kiro/deepseek-3.2` (0.25x) — if Auto doesn't work well
- `kiro/qwen3-coder-next` (0.05x) — if even cheaper needed
