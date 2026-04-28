# permissions_fix

kind: let

---

## Fix: First-Run Permissions Crash (bd-ikz, GitHub #10)

### Problem

When Screen Recording permission hasn't been granted, `SCShareableContent.excludingDesktopWindows` and `SCStream.startCapture()` throw errors that could crash the app or produce opaque error messages. The existing preflight check in `MirrorEngine.start()` caught the obvious case, but `ScreenCapture.start()` itself had no guard, meaning any code path that reached it without the engine-level check would crash.

### Changes

**`Sources/MirrorEngine/MirrorEngine.swift`**

1. Added `ScreenCaptureError` enum with two cases:
   - `.permissionDenied` -- clear message directing user to System Settings > Privacy & Security > Screen Recording
   - `.contentEnumerationFailed(Error)` -- wraps the underlying `SCShareableContent` error with actionable guidance

2. Added `CGPreflightScreenCaptureAccess()` guard at the top of `ScreenCapture.start()`, before any ScreenCaptureKit calls. This is defense-in-depth: even if a caller bypasses `MirrorEngine.start()`, the capture layer itself refuses to proceed without permission.

3. Wrapped `SCShareableContent.excludingDesktopWindows()` in its own do/catch to convert opaque system errors into `ScreenCaptureError.contentEnumerationFailed` with a user-friendly message.

4. Improved error cleanup in `MirrorEngine.start()`: on capture failure, the compositor pacer, servers, and display manager are all torn down (previously the pacer and server references leaked on this path).

**`Sources/App/DaylightMirrorApp.swift`**

5. The "Start Mirror" button in the idle menu bar view now checks `MirrorEngine.hasScreenRecordingPermission()` first. If permission is missing, it opens the setup wizard instead of attempting (and failing) to start capture.

### Defense-in-Depth Layers

The app now has three layers of permission protection:

| Layer | Location | Behavior |
|-------|----------|----------|
| UI gate | Menu bar "Start Mirror" button | Redirects to setup wizard if no permission |
| Engine gate | `MirrorEngine.start()` line 1605 | Calls `CGRequestScreenCaptureAccess()`, sets `.error` status |
| Capture gate | `ScreenCapture.start()` line 715 | Throws `ScreenCaptureError.permissionDenied` |

No code path can reach `SCStream` without passing through at least one of these checks. The app never crashes -- it shows an actionable error message or the setup wizard.

### Build Verification

```
swift build -c release  # Build complete! (4.61s)
```
