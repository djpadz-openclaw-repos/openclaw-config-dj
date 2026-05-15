# FAMOUS_PEERS.md — Multiplayer iOS/tvOS Game

Multiplayer iOS/tvOS game with 110 duos using GameKit and MultipeerConnectivity.

## Technology Stack

- **Platforms:** iOS, tvOS
- **Multiplayer:** GameKit + MultipeerConnectivity
- **Scale:** 110 duos (220 players)

## Key Lessons

### Swift Package Manager Resources (2026-04-11)

**Problem:** Spent ~1 hour debugging resource loading by guessing at different Package.swift configurations.

**Solution:** Resources MUST be in `Sources/[TargetName]/Resources/` for Bundle.module to be synthesized. This is documented in Apple's official docs and discussed extensively on Stack Overflow.

**Correct approach:**
- Place resources in `Sources/[TargetName]/Resources/`
- Access with: `Bundle.module.url(forResource: "filename", withExtension: "ext")`
- This is the correct and recommended way per Apple documentation
- Don't try to access via Bundle(for:) or other workarounds

**Lesson:** When facing a technical problem, search first for authoritative sources before iterating through guesses. See `LESSONS.md` for full details.

## Development Notes

- Uses cards.json for game data
- Multiplayer coordination via GameKit
- Local connectivity via MultipeerConnectivity
