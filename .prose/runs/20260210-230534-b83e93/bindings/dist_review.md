# dist_review
kind: let
---

## Distribution Fixes Review

### 1. Cask Fix (bd-fma) — GOOD

**Cask formula** (`/opt/homebrew/Library/Taps/welfvh/homebrew-tap/Casks/daylight-mirror.rb`):
- Downloads `app-debug.apk` from the correct release URL
- Copies it to `/opt/homebrew/share/daylight-mirror/app-debug.apk`
- Caveats instruct user to `adb install /opt/homebrew/share/daylight-mirror/app-debug.apk`

**README** (`/Users/moritzbierling/werk/wield/daylight-mirror/README.md`):
- Homebrew section (line 36) correctly references `app-debug.apk` at the same path
- DMG/Releases sections still reference `DaylightMirror.apk` (lines 20, 43) — this is correct since the DMG bundles a renamed APK

No issues found.

### 2. Code Signing Fix (bd-1hs) — GOOD

**Makefile** (`/Users/moritzbierling/werk/wield/daylight-mirror/Makefile`), install target (lines 33-42):
- `cp Info.plist` to `Contents/Info.plist` — real file exists at repo root
- `cp Resources/AppIcon.icns` to `Contents/Resources/AppIcon.icns` — file exists at `Resources/AppIcon.icns`
- `codesign --force --deep -s -` ad-hoc signs the bundle

This addresses the Gatekeeper error ("code has no resources but signature indicates they must be present"). The bundle now has a proper Info.plist, icon resource, and valid ad-hoc signature.

No issues found.

### 3. Package Name Fix (bd-39g) — GOOD

Audit confirmed docs already use `com.daylight.mirror`. No changes needed. Correctly closed as no-op.

### Bead Status

All three beads are closed:
- **bd-fma** (Fix Homebrew cask APK download URL) — CLOSED
- **bd-1hs** (Fix Mac app code signing) — CLOSED
- **bd-39g** (Fix Android package name mismatch) — CLOSED

### Items Needing Rework

**None.** All three fixes look correct. The Makefile install target properly constructs a signable bundle, the cask formula downloads the right APK filename, and the README instructions match the actual file paths.
