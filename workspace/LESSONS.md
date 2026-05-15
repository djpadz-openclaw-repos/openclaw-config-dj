# LESSONS.md — Problem-Solving Patterns & Lessons Learned

## Critical Lesson: Research First, Guess Never (2026-04-11)

**What happened:** Spent ~1 hour and massive token budget debugging Famous Peers resource loading by guessing at different Package.swift configurations. Moved cards.json back and forth between locations, tried different bundle loading approaches, all while the actual answer was one Google search away.

**The actual problem:** Swift Package Manager only synthesizes `Bundle.module` when resources are in `Sources/[TargetName]/Resources/`. This is documented in Apple's official docs and discussed extensively on Stack Overflow. I should have found this in 5 minutes.

**The lesson:**
- When facing a technical problem I don't immediately understand: **SEARCH FIRST**
- Look for authoritative sources: Apple docs, official frameworks, Stack Overflow
- Don't iterate through guesses hoping one works
- Guessing wastes tokens (money), time, and erodes trust
- If the user suggests searching or provides a documentation link, act on it immediately

**How to apply this:**
1. If I'm stuck after 1-2 attempts, stop and search
2. Search for: official docs, Stack Overflow, framework discussions
3. Read the actual answer before trying more code changes
4. Only iterate on code after understanding the root cause

**Swift Package resources specifically:**
- Resources MUST be in `Sources/[TargetName]/Resources/` for Bundle.module to be synthesized
- Access them with: `Bundle.module.url(forResource: "filename", withExtension: "ext")`
- This is the correct and recommended way per Apple documentation
- Don't try to access via Bundle(for:) or other workarounds

## Tool Selection (2026-04-05)

- When high-level tools fail (e.g., `web_fetch` with readability extraction), don't just accept the limitation
- Try lower-level alternatives: `exec` + `curl` + text processing tools (`grep`, `sed`, etc.)
- For structured data (tables, JSON, etc.), direct parsing is often more reliable than automated content extraction
- Be proactive about suggesting alternatives instead of waiting for the user to point them out
- Raw HTML parsing beats automated extraction when the page structure is complex or non-standard

## Thoroughness & Verification (2026-04-05, reinforced 2026-04-06)

- **CRITICAL:** Never declare work "complete" or "verified" without doing a comprehensive search
- Don't do partial searches and assume the rest is done - search everything at once
- Be honest about what you've actually verified vs. what you're assuming
- Don't make the user do your work by manually pointing out files you missed
- When migrating paths/configs across a system, do ONE comprehensive search across ALL directories, not multiple partial searches
- Overconfidence in incomplete verification wastes the user's time and erodes trust
- Always verify with a final comprehensive search before declaring anything complete
- **Apr 6 lesson:** User caught me claiming things were "complete" and "clean" multiple times when they weren't (5 projects without .git, 20+ workspace references, 9 more references after "final" verification). This was frustrating and eroded trust. When the user is skeptical ("Are you _sure_?"), they're right to be — verify again. Systematic verification beats confident guessing every time.

## Cron Job Defaults

- **Always use `lightContext: true`** for new isolated cron jobs unless the job specifically needs full personal context. Simple command-running jobs, stock checks, health checks, etc. should all use lightContext.
- Default model for all cron jobs: `kiro/claude-haiku-4.5`.
