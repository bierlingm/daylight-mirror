# bundle_apk

kind: let

---

## Summary

Added automatic APK bundling and installation to Daylight Mirror. When the Mac app detects a connected Daylight device, it checks whether the companion Android app is installed. If not, it installs the bundled APK automatically with progress shown via the `apkInstallStatus` published property.

## Changes

### `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/MirrorEngine/MirrorEngine.swift`

**ADBBridge -- two new static methods:**

1. `isAppInstalled() -> Bool` -- Runs `adb shell pm list packages com.daylight.mirror` and checks for the package in output.
2. `installBundledAPK() -> String?` -- Looks for `app-debug.apk` in `Bundle.main.resourcePath`, runs `adb install -r <path>`. Returns nil on success or an error string on failure.

**MirrorEngine -- new published property:**

- `@Published public var apkInstallStatus: String = ""` -- Empty when idle, "Installing companion app..." during install, "Installed" on success, or error message on failure. UI can bind to this for menu bar status.

**MirrorEngine.start() -- install logic added to step 4b:**

Before setting up the reverse tunnel, the engine now:
1. Calls `ADBBridge.isAppInstalled()` to check if the companion app exists
2. If not installed, sets `apkInstallStatus` and calls `ADBBridge.installBundledAPK()`
3. Logs success or error, then proceeds with tunnel setup and app launch as before
4. Clears `apkInstallStatus` on successful connection

### `/Users/moritzbierling/werk/wield/daylight-mirror/Makefile`

**`install` target updated** to copy the APK into the app bundle:

```makefile
@# Embed companion APK if available (run 'make android' first)
@if [ -f "$(APK)" ]; then \
    cp "$(APK)" "$(APP_BUNDLE)/Contents/Resources/app-debug.apk"; \
    echo "Embedded companion APK"; \
else \
    echo "No APK found (run 'make android' to build). Auto-install will be skipped."; \
fi
```

The APK is placed at `Contents/Resources/app-debug.apk` in the bundle, matching the path `installBundledAPK()` looks for.

## Build verification

`swift build -c release` passes cleanly.

## Build steps for full bundle

```bash
# 1. Build the Android APK (requires Android SDK)
make android

# 2. Download adb (one-time)
make fetch-adb

# 3. Build + install Mac app with embedded adb + APK
make install
```

The resulting `~/Applications/Daylight Mirror.app/Contents/Resources/` will contain:
- `adb` -- the embedded ADB binary
- `app-debug.apk` -- the companion Android app

## Error handling

- **No bundled APK**: `installBundledAPK()` returns early with "No bundled APK found". The app continues without installing -- the user can install manually via `make deploy`.
- **ADB not available**: Returns "ADB not available". No crash.
- **Install failure** (device not in debug mode, insufficient storage, etc.): The `adb install` stderr/stdout is captured and surfaced in `apkInstallStatus` and logged. The app still attempts tunnel setup and launch in case the app was installed some other way.
- **App already installed**: `isAppInstalled()` returns true, entire install block is skipped. Zero overhead on subsequent launches.

## What's NOT included (by design)

- No actual APK binary is committed to git
- No version comparison (always skips if package exists; future work could compare versionCode)
- No UI changes to display `apkInstallStatus` in the menu bar (UI binding is ready; SwiftUI view update is a separate task)
