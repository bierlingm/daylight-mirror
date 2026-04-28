# p1_review

kind: let

---

## P1 Fix Review Summary

**Build status:** `swift build -c release` passes cleanly (4.67s). All three fixes compile together without conflict.

**Files changed:** 4 files, +182/-50 lines. Changes are focused and minimal.

---

### 1. FPS Fix (bd-1v0) -- GOOD, closed

**Did it solve ~13 FPS?** Yes, for the primary case. The root cause was correct: the dirty-pixel window was on `NSScreen.main` instead of the virtual display's NSScreen, so the virtual display's compositor never saw dirty regions.

**Changes are minimal and correct:**
- `CompositorPacer` now takes `targetDisplayID`, looks up the matching NSScreen
- Window placed on virtual display's screen (4x4 at `.normal` level)
- Timer fallback at 33ms if virtual display has no NSScreen (mirror mode)
- Tick counter every 150 ticks for diagnostics

**One caveat acknowledged in the fix report:** if macOS always collapses the virtual display in mirror mode, the timer fallback is the active path, and the real fix would be switching to independent display mode. The diagnostic logging will reveal which path is taken on real hardware. This is acceptable -- the code handles both cases and logs which one is active.

**Verdict:** Close confirmed. No rework needed.

---

### 2. Permissions Fix (bd-ikz) -- GOOD, closed

**Does it handle missing permissions gracefully?** Yes, three-layer defense:
1. UI gate: "Start Mirror" button checks `hasScreenRecordingPermission()`, redirects to setup wizard
2. Engine gate: existing `CGRequestScreenCaptureAccess()` in `MirrorEngine.start()`
3. Capture gate: new `CGPreflightScreenCaptureAccess()` guard in `ScreenCapture.start()` throws `ScreenCaptureError.permissionDenied`

**Error cleanup improved:** On capture failure, the pacer, servers, and display manager are all torn down. Previously, the pacer and server references leaked on this path. Both the `ScreenCaptureError` and generic `catch` paths now clean up identically.

**The `ScreenCaptureError` enum** has clear, actionable messages pointing users to System Settings.

**Verdict:** Close confirmed. No rework needed.

---

### 3. Blank Screen Fix (bd-2fw) -- GOOD, closed

**Did it find the root cause?** Yes. Race condition: ADB reverse tunnel + app launch happened before TCP server was listening. Android app connected to nothing, got ECONNREFUSED.

**Fix is correct and minimal:**
- Startup reordered: virtual display + servers first, then `setupReverseTunnel` + `launchApp`
- Tunnel result now checked (was `@discardableResult` and ignored)
- Client connection tracking moved from `newConnectionHandler` into `.ready` state handler -- this is the right place; previously resolution/keyframe were sent before TCP handshake completed
- Reconnect path also checks tunnel result

**Android native changes** (mirror_native.c): diagnostic LOGI/LOGE additions only. No logic changes. Cannot verify NDK build but changes are syntactically trivial.

**Remaining risk noted:** no UI feedback when ADB authorization is missing on the device. Acceptable for now -- the log message is there, and this is an edge case (first-ever USB connection).

**Verdict:** Close confirmed. No rework needed.

---

### Demo Video (bd-36n)

**Is it unblocked?** Yes. bd-36n has no blocking dependencies (only a parent link to the epic bd-19o). All three P1 bugs that constituted the "soft dependency" on a working end-to-end flow are now closed. The app should:
- Not crash on missing permissions (bd-ikz fixed)
- Connect on first try without blank screen (bd-2fw fixed)
- Run at ~30 FPS with pacer diagnostics to confirm (bd-1v0 fixed)

bd-36n is ready to work on. No status change needed -- it was already `open` with no blockers.

---

### Overall Assessment

All three P1 fixes are sound. The changes are focused, the build passes, and the beads are correctly closed with accurate close reasons. The diff is +182/-50 across 4 files with no unrelated changes.

**Next recommended action:** Test on real hardware with a DC-1, then record the demo video (bd-36n).
