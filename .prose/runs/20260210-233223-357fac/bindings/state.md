# state

kind: let

---

## Repository State: Daylight Mirror (2026-02-10)

### Git History (Sprint 1 Merged to main)

All Sprint 1 work has been merged. The commit log shows a clear progression:

1. **beaf796** Menu bar app, resolution picker, display controls, graceful reconnect
2. **068a4c9** Fix warmth control, add restart button, pulsing reconnect overlay
3. **4486817** Fix warmth: revert to ADB path (protected Daylight setting)
4. **e8f82a9** Add F-key shortcut reference and GitHub link to menu bar
5. **ea74685** Add Ctrl+F8 shortcut pill
6. **a2a14ca** Add in-app update checker, resolution docs, keyboard shortcut reference
7. **7338709** Auto-dim Mac when Daylight connects, quadratic brightness curve
8. **ab81c45** Add app icon, permissions check, first-run docs
9. **62a51bb** Add setup wizard, fix Android flicker, Developer ID signing
10. **55e24dd** Add full CLI control via Unix domain socket IPC
11. **fddd5d6** Add Cozy (800x600) resolution, always-on control socket
12. **975ce20** Bump version to 1.2.0
13. **224fd23** HiDPI Cozy, sharpening slider, expanded stats
14. **a5f468b** Update README for v1.3 (HEAD)

Current version: **v1.3** (latest tag). The app is a macOS menu bar app with a Swift Package Manager build (`server/` directory), plus an Android companion APK (`android/` directory).

### Codebase Structure

- **MirrorEngine.swift** (1769 lines): Core engine. Handles virtual display creation, SCScreenCaptureKit streaming, TCP/WS/HTTP servers, greyscale conversion (vImage SIMD), LZ4 delta compression, display controls (brightness/warmth/backlight via ADB), and a CompositorPacer for 30 FPS.
- **Makefile**: `make install` builds via `swift build -c release` and bundles into `~/Applications/Daylight Mirror.app`. `make deploy` pushes APK via ADB. Version in Info.plist is hardcoded to 1.0 (stale).
- **Resolution presets**: Cozy (HiDPI 2x, 800x600pt/1600x1200px), Comfortable (1024x768), Balanced (1280x960), Sharp (1600x1200 native).

### Beads: All Issues (14 total)

#### In Progress (1)
| ID | P | Title |
|----|---|-------|
| bd-1v0 | 1 | Achieve advertised 30 FPS (CompositorPacer implemented, needs validation) |

#### Ready to Work On (11 items from `br ready`)

**Priority 1 (Critical):**
| ID | Title | Notes |
|----|-------|-------|
| bd-19o | Daylight Mirror 10x Improvement | Epic (parent of all 20 children). No blockers. |
| bd-ikz | Improve first-run permissions flow | GitHub #10. Crash on missing screen recording permission. |
| bd-2fw | Debug blank screen on DC-1 after setup | GitHub #11. DC-1 stays blank, Mac says "Waiting for client". |
| bd-36n | Record demo video | Needs working end-to-end flow first (soft dependency). |

**Priority 2 (Important):**
| ID | Title | Notes |
|----|-------|-------|
| bd-2do | Ship binaries for all tagged releases | v1.1 and v1.2 missing binaries. GitHub Actions workflow drafted. |
| bd-2z7 | Bundle APK inside Mac app | Zero-friction install of companion APK. |
| bd-3aj | Zero-config USB detection and streaming | Auto-detect DC-1, no terminal needed. |

**Priority 3 (Nice to have):**
| ID | Title | Notes |
|----|-------|-------|
| bd-1nn | Add Android app icon | GitHub #9. No icon in DC-1 launcher. |
| + 3 more | (connection status, auto-reconnect, portrait) | |

#### Not Ready (blocked, 0 items)
All issues with dependencies are children of the epic bd-19o via parent links, but those links don't create blockers -- they're structural. No issue is currently blocked by another open issue.

### Recommended Work Order

**Immediate (highest impact, no blockers):**

1. **bd-ikz** -- Improve first-run permissions flow (P1 bug, GitHub #10). Prevents crash on first launch. Ship-blocking.
2. **bd-2fw** -- Debug blank screen on DC-1 (P1 bug, GitHub #11). Core functionality broken for new users.
3. **bd-1v0** -- Achieve 30 FPS (P1, in progress). CompositorPacer is implemented but unvalidated.

**Next batch:**

4. **bd-2do** -- Ship binaries for tagged releases (P2). GitHub Actions workflow is already drafted. Quick win.
5. **bd-1nn** -- Add Android app icon (P3). Small, self-contained.

**Deferred (require architectural work):**

6. **bd-3aj** -- Zero-config USB detection (P2). Needs bd-1dy (embed ADB) first in practice.
7. **bd-2z7** -- Bundle APK (P2). Also needs embedded ADB.
8. **bd-1dy** -- Embed minimal ADB binary (P2). Enables zero-config and bundled APK.

### Key Observations

- **No CI/CD exists.** All releases are manual. The release workflow (bd-2do) is the highest-leverage infrastructure task.
- **Version drift.** Makefile hardcodes Info.plist version to 1.0 while the actual release is v1.3. Should be fixed alongside release automation.
- **Two P1 bugs from external users** (GitHub #10 and #11) are the most urgent user-facing issues. Both relate to first-run experience.
- **FPS work (bd-1v0)** is in progress with CompositorPacer already landed in code. Needs real-device testing to close.
- **The dependency graph is flat.** All items are children of the epic but don't block each other. Work can proceed in parallel on any ready item.
