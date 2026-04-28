# adb_embed

kind: let

---

## Summary

Embedded ADB binary support in Daylight Mirror. The app now checks for a bundled `adb` at `Bundle.main.resourcePath/adb` before falling back to PATH lookup. No binary is committed to git; instead, `make fetch-adb` downloads platform-tools on demand, and `make install` copies the binary into the .app bundle if present.

## Changes

### `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/MirrorEngine/MirrorEngine.swift`

**ADBBridge rewritten** to resolve the adb path once at startup:

1. `resolvedADBPath` (lazy static) checks `Bundle.main.resourcePath + "/adb"` first, then falls back to `which adb` on PATH. Logs which binary is selected.
2. `makeADBProcess(_:)` factory creates a `Process` pointing directly at the resolved binary (no more `/usr/bin/env adb` indirection).
3. All 7 ADB methods (`connectedDevice`, `setupReverseTunnel`, `removeReverseTunnel`, `querySystemSetting`, `setSystemSetting`, `launchApp`, `isAvailable`) now use `makeADBProcess` and gracefully return nil/false if no adb is found.

### `/Users/moritzbierling/werk/wield/daylight-mirror/Makefile`

- **`fetch-adb` target**: Downloads `platform-tools-latest-darwin.zip` from Google, extracts to `tools/platform-tools/`.
- **`install` target updated**: Creates `Contents/Resources/` in the app bundle. If `tools/platform-tools/adb` exists, copies it to `Contents/Resources/adb` with +x. Otherwise prints a note that system adb will be used.

### `/Users/moritzbierling/werk/wield/daylight-mirror/.gitignore`

- Added `tools/` to prevent committing the downloaded platform-tools.

## Build verification

`swift build -c release` passes cleanly after all changes.

## How to use

```bash
# One-time: download adb (~6MB)
make fetch-adb

# Build + install with embedded adb
make install

# The app will log which adb it uses:
# [ADB] Using bundled adb: /Users/.../Daylight Mirror.app/Contents/Resources/adb
# or
# [ADB] Using system adb: /opt/homebrew/bin/adb
```

## What's NOT included (by design)

- No actual adb binary is committed to git
- No notarization changes (the bundled adb is an unsigned Google binary; for distribution, the entire .app needs Developer ID signing which will cover embedded binaries)
- No automatic download during `make install` -- user must explicitly run `make fetch-adb` first
