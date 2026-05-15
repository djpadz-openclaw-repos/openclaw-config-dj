# CARPLAY_GIZMO.md — OpenClaw iOS Companion App

Native iOS CarPlay app for natural voice conversation with Gizmo, your OpenClaw AI assistant.

## What It Does

- **Push-to-talk voice interface** in CarPlay (tap mic → speak → tap again → hear response)
- **Speech-to-text** via `SFSpeechRecognizer` + `AVAudioEngine`
- Sends messages to OpenClaw gateway with **full conversation history** (multi-turn)
- **SSE streaming** — tokens appear live in CarPlay display as they arrive (falls back gracefully if gateway doesn't support it)
- **TTS** via `AVSpeechSynthesizer` (system) or **ElevenLabs** (high-quality AI voices) — swappable at runtime
- **Interrupt at any time** — tap mic during processing or speaking to stop and start over
- **Clear history** button (trash icon)
- Error states with 4-second auto-recovery
- Minimal iOS companion view showing live connection status, state, and streaming buffer

## Current Status

- **Development:** In progress
- **CarPlay entitlement:** Not yet approved by Apple (coding around it for now)
- **MVP:** Core voice conversation working, ready for testing
- **Architecture:** Modular, state machine-based conversation flow

## Architecture

```
CarPlayGizmo/
├── App/
│   ├── CarPlayGizmoApp.swift          # @main entry
│   ├── AppDelegate.swift              # Scene registration, permissions
│   └── ContentView.swift             # iOS companion: status, state, streaming buffer
├── CarPlay/
│   ├── CarPlaySceneDelegate.swift     # CPTemplateApplicationSceneDelegate
│   └── GizmoCarPlayTemplateManager.swift  # State machine + template lifecycle
└── Shared/
    ├── OpenClawClient.swift           # HTTP client — request/response + SSE streaming, history
    ├── SpeechRecognitionManager.swift # AVAudioEngine + SFSpeechRecognizer (push-to-talk)
    ├── TTSManager.swift               # TTSProvider protocol + AVSpeechSynthesizer default
    ├── MicrosoftTTSProvider.swift     # Azure Cognitive Services neural TTS (preferred)
    ├── ElevenLabsTTSProvider.swift    # ElevenLabs TTS (fallback)
    └── AudioSessionManager.swift      # AVAudioSession routing utilities
```

## Conversation State Machine

```
idle ──[tap mic]──► listening ──[tap mic]──► processing ──► speaking ──► idle
  ▲                                              │               │          │
  └──────────────────────[tap mic to interrupt]──┴───────────────┘          │
  └────────────────────────────────[auto on complete]────────────────────────┘
  └────────────────────[auto after 4s]── error ◄── (any failure)
```

## TTS Providers

| Provider | Quality | Latency | Setup |
|----------|---------|---------|-------|
| `MicrosoftTTSProvider` | Excellent | ~300ms | Azure Speech key + region |
| `ElevenLabsTTSProvider` | Excellent | ~400ms | ElevenLabs API key + voice ID |
| `AVFoundationTTSProvider` | Good | ~0ms | Built-in, no config needed |

## Roadmap / TODO

- [ ] **VAD (Voice Activity Detection)** — auto-stop on silence (no second tap needed)
- [ ] **App icon** — add a 1024×1024 Gizmo/fox icon to Assets.xcassets
- [ ] **`CPInformationTemplate`** — scrollable conversation history view when parked
- [ ] **Wakeword** — "Hey Gizmo" hands-free trigger (Picovoice Porcupine or iOS SFSpeechRecognizer background task)
- [ ] **Mock URLSession** — unit tests for OpenClawClient network logic
- [ ] **Sign in with Apple** — auth for multi-user/cloud history sync
- [ ] **Haptic feedback** — iOS-side vibration on state transitions (not available in CarPlay itself)

## Repository

- **GitHub:** https://github.com/djpadz-openclaw-repos/carplay-gizmo
- **Setup:** See README.md in repo for detailed setup instructions
- **Requirements:** Xcode 15+, XcodeGen, Apple Developer account (paid)
