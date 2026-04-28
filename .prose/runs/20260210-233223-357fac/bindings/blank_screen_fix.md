# blank_screen_fix

kind: let

---

## Root Cause: ADB Reverse Tunnel Race Condition

The primary cause of the blank screen (bd-2fw, GitHub #11) is a **startup ordering bug**: the ADB reverse tunnel and Android app launch happened BEFORE the TCP server started listening.

### Sequence before fix:
1. `ADBBridge.setupReverseTunnel(port: 8888)` -- forwards device:8888 to Mac:8888
2. `ADBBridge.launchApp()` -- Android app starts, native code connects to 127.0.0.1:8888
3. ... virtual display creation (1s sleep) ...
4. ... mirroring setup (1s sleep) ...
5. `TCPServer.start()` -- NOW listening on port 8888

The Android native code connects during step 2-3, but nothing is listening on Mac port 8888 yet. Connection fails with ECONNREFUSED. Native code retries after 1s. Meanwhile on the Mac side, the `onClientCountChanged` callback never fires, so the UI stays at "Waiting for client."

If the retry happens to land during steps 3-4 (before the server is ready), it fails again. Eventually it connects, but the user experience is a 2-4 second blank period with no feedback, and in some cases the tunnel itself can fail silently (ADB not in PATH, device authorization expired, etc.).

### Secondary issue: premature client tracking

The `TCPServer.newConnectionHandler` was adding connections to the pool and firing `onClientCountChanged` before the NWConnection reached `.ready` state. Resolution and keyframe were also sent before the connection was fully established.

## Changes Made

### 1. Reordered startup: ADB tunnel AFTER server is listening

**File:** `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/MirrorEngine/MirrorEngine.swift`

Moved `setupReverseTunnel` and `launchApp` from before virtual display creation to after `tcp.start()`. Now the sequence is:

1. Virtual display + mirroring
2. TCP/WS/HTTP servers start
3. `setupReverseTunnel` (target port is now listening)
4. `launchApp` (Android connects immediately on first try)

### 2. Check and log tunnel result

The `setupReverseTunnel` return value was previously discarded (`@discardableResult`). Now both `start()` and `reconnect()` check the return value and log success/failure. If the tunnel fails, `adbConnected` is set to `false` and a warning is printed.

### 3. Fixed client connection tracking to fire at `.ready`

Moved connection append, `onClientCountChanged`, `sendResolution`, and cached keyframe send from `newConnectionHandler` into the `.ready` case of `stateUpdateHandler`. This ensures:
- Client count is accurate (only counts fully established connections)
- Resolution and keyframe are sent over a ready connection
- Mac UI transitions from "Waiting for client" only when the TCP handshake is complete

### 4. Added diagnostic logging in native code

**File:** `/Users/moritzbierling/werk/wield/daylight-mirror/android/app/src/main/cpp/mirror_native.c`

- Log each connection attempt (not just failures)
- Log "is ADB reverse tunnel set up?" hint on connect failure
- Log first frame rendered with resolution, type (keyframe/delta), and compressed size
- Log if frame arrives but `g_window` is NULL (surface not ready)

## Verification

Swift build succeeds cleanly. Android native changes are C-only (no build verification without NDK on this machine, but changes are syntactically correct additions of LOGI/LOGE calls).

## Remaining risk

If the user has ADB installed but not authorized on the device (first USB connection without accepting the debug dialog), `setupReverseTunnel` will fail silently (returns false now, but the user gets no UI indication beyond "Waiting for client"). A future improvement would surface the `adbConnected = false` state in the Mac menu bar UI as "ADB tunnel failed -- check USB authorization."
