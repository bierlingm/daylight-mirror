# flicker_fix

kind: let

---

## Connection State Flicker Fix (bd-2x2, GitHub #13)

### Problem

The status overlay on Android flickered during reconnection. The native C layer (`mirror_native.c`) disconnects and reconnects with a 1-second sleep between attempts (line 399). The previous 500ms debounce in `MirrorActivity.onConnectionState()` was shorter than the reconnect cycle, so the overlay would flash visible during every transient disconnect.

### Root Cause

In `mirror_native.c`, the reconnect loop is:
1. Socket closes (connection lost)
2. `onConnectionState(false)` fires immediately
3. `sleep(1)` -- native thread waits 1 second
4. Reconnect attempt; if successful, `onConnectionState(true)` fires

With a 500ms debounce, the overlay appeared at 500ms and disappeared at ~1000ms -- a visible flash every reconnect cycle.

### Fix Applied

**File**: `/Users/moritzbierling/werk/wield/daylight-mirror/android/app/src/main/java/com/daylight/mirror/MirrorActivity.kt`

Changed `onConnectionState()` to use a three-tier state machine:

| Transition | Delay | UI |
|---|---|---|
| connected -> disconnected | 2s debounce | Nothing visible (absorbed) |
| 2s elapsed, still disconnected | immediate | "Reconnecting..." (title only, no hints) |
| 10s total disconnected | immediate | Full "Waiting for Mac..." with step-by-step hints |
| disconnected -> connected | 0ms | Hide overlay instantly |

Key changes:
1. **Debounce increased from 500ms to 2000ms** -- covers the 1s native reconnect cycle with margin
2. **Minimal reconnecting state** -- hides `statusHint` and shows only "Reconnecting..." title, avoiding the jarring full-screen instruction overlay for brief outages
3. **Escalation after 8 more seconds** -- if still disconnected after 10s total (2s debounce + 8s escalation), shows the full waiting screen with cable/setup instructions

### Build Verification

No JDK installed on this machine. The change compiles against the same Android SDK APIs already in use (`View.GONE`, `Handler.postDelayed`, `TextView.visibility`). No new imports or dependencies. Build must be verified on a machine with the Android toolchain:

```bash
cd ~/werk/wield/daylight-mirror/android && ./gradlew assembleDebug
```

### What Was NOT Changed

- **mirror_native.c** -- The C reconnect loop timing (1s sleep) is correct. The problem was purely on the UI debounce side.
- **Initial "Waiting for Mac..." screen** -- Still shown on app launch before first connection (no debounce needed for that path).
