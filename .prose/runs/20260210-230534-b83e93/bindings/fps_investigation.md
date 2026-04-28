# fps_investigation
kind: let
---

## Finding: The dirty-pixel workaround is already implemented

The `CompositorPacer` class (lines 593-652 of `Sources/MirrorEngine/MirrorEngine.swift`) already implements the exact dirty-pixel trick described in the issue. It is already wired into the engine lifecycle.

### What exists

**CompositorPacer** (line 593):
- Creates a 1x1 `NSWindow` at `(0, screen.maxY - 1)` (hidden under the menu bar)
- Borderless, no shadow, ignores mouse events, level `.screenSaver + 1`
- Uses `CADisplayLink` (macOS 14+) at preferred 30Hz
- Toggles background between `#000000` and `#010000` (1/255 red channel diff) each tick
- Imperceptible on any display

**Integration in MirrorEngine.start()** (line 1552-1556):
```swift
let pacer = CompositorPacer()
pacer.start()
compositorPacer = pacer
```

**Teardown in MirrorEngine.stop()** (line 1673-1674):
```swift
compositorPacer?.stop()
compositorPacer = nil
```

### Why it should work

The virtual display is configured as a **mirror** of the built-in display (`CGConfigureDisplayMirrorOfDisplay`). Any dirty region on the main screen forces WindowServer to recomposite the mirrored output too. The 1x1 pixel toggle at 30Hz should prevent the compositor from considering the frame "clean" and skipping SCStream delivery.

### Potential issues if FPS is still low

1. **CADisplayLink preferred rate vs actual**: The `preferredFrameRateRange` is set to min=30, max=60, preferred=30. On some configurations, macOS may not honor this for a display link tied to `NSScreen.main` when the virtual display is the capture target. The display link fires based on the main display's vsync, not the virtual display's.

2. **Window not on virtual display**: The dirty pixel window is placed on `NSScreen.main`. If macOS optimizes per-display compositing, the dirty region on the built-in display may not force recompositing on the virtual display. Since mirroring means both displays share the same backing buffer, this *should* be fine, but it's worth verifying empirically.

3. **SCStream `minimumFrameInterval` vs compositor rate**: SCStream is configured with `minimumFrameInterval = CMTime(1, 30)` (line 732). Even if the compositor runs at 60Hz, SCStream caps at 30fps. If the compositor only recomposites at ~13Hz despite the dirty pixel, the issue is upstream of SCStream.

4. **Window level too high**: `.screenSaver + 1` is extremely high. If SCStream's content filter excludes windows above a certain level, the dirty pixel may not appear in the captured content. But since it's on the main display (not directly captured), this shouldn't matter for the mirror path.

### Recommendation

The implementation looks correct. If FPS is still ~13 despite the pacer:

1. **Verify the pacer is actually running**: Add a frame counter to `tick()` and log every 5 seconds.
2. **Try placing the dirty window on the virtual display**: Instead of `NSScreen.main`, find the `NSScreen` matching `displayManager.displayID` and position the window there.
3. **Try a larger dirty region**: 1x1 may be below some threshold. Try 2x2 or 4x4.
4. **Try lower window level**: Use `.normal` instead of `.screenSaver + 1` to ensure it's in the normal compositing path.
5. **Try DispatchSourceTimer instead of CADisplayLink**: CADisplayLink may not fire reliably for non-visible displays.

### Status

No code changes made. The dirty-pixel workaround already exists and is integrated. The bead should be updated to reflect this -- the fix is deployed, and if FPS is still low, the issue is elsewhere (possibly macOS version-specific behavior or mirror-mode compositor optimization that the dirty pixel doesn't circumvent).
