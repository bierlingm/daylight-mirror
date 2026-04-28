# cask_fix
kind: let
---

## Fixed APK URL in Homebrew cask formula

**Problem**: The cask's postflight script downloaded `DaylightMirror.apk` from the GitHub release, but the actual asset is named `app-debug.apk`. This caused a silent 404 during `brew install`, leaving users without the Android APK.

**Files changed**:

1. `/opt/homebrew/Library/Taps/welfvh/homebrew-tap/Casks/daylight-mirror.rb` -- All three references to `DaylightMirror.apk` replaced with `app-debug.apk`:
   - curl download URL (line 20)
   - staged_path filename (line 19)
   - cp destination and caveats instruction (lines 24, 34)

2. `/Users/moritzbierling/werk/wield/daylight-mirror/README.md` -- Updated the Homebrew install instructions from `DaylightMirror.apk` to `app-debug.apk`.

**Note**: The cask is still pinned to `version "1.0"` and the Mac zip SHA matches v1.0. The Mac app URL (`DaylightMirror-1.0-mac.zip`) was not changed as it was not reported broken. Updating the cask version to v1.3 is a separate task.

**Bead closed**: `bd-fma`
