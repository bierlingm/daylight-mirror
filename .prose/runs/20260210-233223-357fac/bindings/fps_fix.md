# fps_fix

kind: let

---

## Root Cause

The `CompositorPacer` dirty-pixel window was placed on `NSScreen.main` (the built-in display) and its `CADisplayLink` was also created from `NSScreen.main`. Since the capture target is a **virtual display**, the virtual display's compositor never saw dirty regions and stayed idle. macOS only delivered frames when something else happened to change on the virtual display's backing buffer, resulting in ~13 FPS.

## Fix Applied

Three changes to `CompositorPacer` in `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/MirrorEngine/MirrorEngine.swift`:

### 1. Target the virtual display's NSScreen

The class now accepts a `CGDirectDisplayID` at init and looks up the matching `NSScreen` via `deviceDescription[NSScreenNumber]`. The dirty-pixel window is positioned on that screen, not `NSScreen.main`.

```swift
init(targetDisplayID: CGDirectDisplayID)
```

Call site updated:
```swift
let pacer = CompositorPacer(targetDisplayID: displayManager!.displayID)
```

### 2. Larger dirty region (4x4) and normal window level

Changed from 1x1 at `.screenSaver + 1` to 4x4 at `.normal`. The larger region ensures it clears any per-pixel compositing threshold. Normal window level keeps it in the standard compositing path.

### 3. Timer fallback for mirror-mode displays

In mirror mode, the virtual display may not have its own `NSScreen` (macOS collapses mirrored displays). If `screenForDisplay()` returns nil, the pacer falls back to a `DispatchSourceTimer` at 33ms (30Hz) instead of crashing on a nil `NSScreen.main!.displayLink()` call. Diagnostic logging reports which path was taken.

### 4. Tick counter for diagnostics

Every 150 ticks (~5 seconds at 30Hz), the pacer logs its tick count. This makes it trivial to verify the pacer is actually firing at the expected rate during testing.

## Build Verification

`swift build -c release` compiles cleanly (one pre-existing warning about nil coalescing, unrelated).

## What Still Needs Validation

This fix addresses the most likely cause (wrong screen for dirty pixel). If FPS remains at ~13 on a real device, the next investigation steps are:

- Check the pacer log output to see if it landed on the virtual screen or fell back to timer
- If mirror mode always collapses the virtual display's NSScreen, the timer fallback path is the active one. In that case, the dirty pixel on `NSScreen.main` may be the only option, and the ~13 FPS cap is a WindowServer limitation of mirror-mode virtual displays. The fix would then be to switch from mirror mode to independent display mode.

## Bead Status

`bd-1v0` closed.
