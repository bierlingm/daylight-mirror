# triage

kind: let

---

## P1 Bug Triage (2026-02-10)

All four P1 items are ready (zero open blockers). bd-1v0 was already in_progress; bd-ikz has been marked in_progress as the first item to tackle.

### Execution Order

| Order | ID | Title | Status | Rationale |
|-------|----|-------|--------|-----------|
| 1 | bd-ikz | Improve first-run permissions flow | **in_progress** | Crash on missing Screen Recording permission. Blocks all first-run testing. Smallest scope of the three code bugs. |
| 2 | bd-2fw | Debug blank screen on DC-1 after setup | open | Core functionality broken for new users. Requires working permissions flow (bd-ikz) to reproduce cleanly. |
| 3 | bd-1v0 | Achieve advertised 30 FPS | **in_progress** | CompositorPacer landed but unvalidated. Needs real-device measurement. Can run in parallel with bd-2fw if a DC-1 is available. |
| 4 | bd-36n | Record demo video | open (deferred) | Soft-blocked by all three above. No point recording until the app works end-to-end at target frame rate. Skip until 1-3 are closed. |

### Why This Order

1. **bd-ikz first**: The app crashes before anything else can happen if permissions are missing. Every tester hits this on first launch. Fixing it unblocks manual testing of bd-2fw and bd-1v0.

2. **bd-2fw second**: Once permissions work, the blank-screen bug is the next wall. This is likely a connectivity/ADB-forwarding issue or missing user instructions. Debugging it requires a working first-run flow.

3. **bd-1v0 third (parallel)**: The dirty-pixel pacer is already in code. This is a validation task -- measure actual FPS on device, tune if needed. Can overlap with bd-2fw work since they touch different parts of the stack (capture pipeline vs. connection setup).

4. **bd-36n last**: Recording a demo of a broken app is pointless. Once the other three are resolved, recording is a 30-minute task.

### Current State

- **bd-ikz**: in_progress (marked now)
- **bd-1v0**: in_progress (marked previously)
- **bd-2fw**: open, next after bd-ikz
- **bd-36n**: open, deferred
