# portrait

kind: let

---

## Portrait Orientation Support (bd-1u8, GitHub #12)

### Summary

Added three portrait (vertical) resolution presets to Daylight Mirror. The DC-1's native panel is 1600x1200; rotated 90 degrees, portrait native is 1200x1600.

### Changes

**`/Users/moritzbierling/werk/wield/daylight-mirror/Sources/MirrorEngine/MirrorEngine.swift`**

Added three new cases to `DisplayResolution`:

| Case | Raw Value | Pixel Dimensions | Notes |
|------|-----------|-------------------|-------|
| `portraitCozy` | `600x800` | 1200x1600 | HiDPI 2x, large UI, native sharpness |
| `portraitBalanced` | `960x1280` | 960x1280 | Good balance of size and sharpness |
| `portraitSharp` | `1200x1600` | 1200x1600 | 1:1 native, maximum sharpness |

Updated computed properties: `width`, `height`, `label`, `isHiDPI` all handle the new cases. Added `isPortrait` computed property for downstream use.

Updated the `ControlSocket` RESOLUTION handler to normalize hyphens to spaces when matching, so CLI users can type `portrait-cozy` and it matches label "Portrait Cozy". Error messages now show hyphenated names (e.g., `portrait-cozy`) for CLI friendliness.

**`/Users/moritzbierling/werk/wield/daylight-mirror/Sources/Mirror/main.swift`**

Updated help text to list portrait presets alongside landscape ones.

### No Android Changes Needed

The Android native renderer (`mirror_native.c`) already handles dynamic resolution via `CMD_RESOLUTION` packets. When a resolution command arrives, it reads `w` and `h` from the packet, reallocates frame buffers, and calls `ANativeWindow_setBuffersGeometry` with the new dimensions. Portrait frames (1200x1600) will be handled identically to any other resolution.

### GUI Auto-Discovery

The SwiftUI menu bar app uses `ForEach(DisplayResolution.allCases)` for its resolution picker. The new portrait cases appear automatically -- no GUI code changes were needed.

### Build Verification

`swift build -c release` completes cleanly with all changes.

### CLI Usage

```
daylight-mirror resolution portrait-cozy
daylight-mirror resolution portrait-balanced
daylight-mirror resolution portrait-sharp
daylight-mirror resolution 1200x1600   # raw value also works
```
