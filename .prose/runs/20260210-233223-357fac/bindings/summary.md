# summary

kind: output

---

## Daylight Mirror -- Sprint 2 Status Report

**Date:** 2026-02-10
**Branch:** main
**Build status:** `swift build -c release` passes cleanly

---

### Bead Inventory (21 total)

| ID | Status | Pri | Type | Title |
|----|--------|-----|------|-------|
| bd-19o | **open** | p1 | feature | Daylight Mirror 10x Improvement (epic) |
| bd-36n | **open** | p1 | task | Record demo video of working Mac-to-DC1 mirroring |
| bd-1dy | **open** | p2 | feature | Embed minimal ADB binary |
| bd-1v0 | closed | p1 | bug | Achieve advertised 30 FPS |
| bd-ikz | closed | p1 | bug | Improve first-run permissions flow |
| bd-2fw | closed | p1 | bug | Debug blank screen on DC-1 after setup |
| bd-3ae | closed | p1 | bug | Fix Mac app code signing for distribution |
| bd-39g | closed | p1 | bug | Fix Android package name mismatch |
| bd-1hs | closed | p1 | bug | Fix Mac app code signing |
| bd-fma | closed | p1 | bug | Fix Homebrew cask APK download URL |
| bd-19h | closed | p1 | feature | Daylight Mirror 10x Improvement (original) |
| bd-2x2 | closed | p2 | bug | Fix connection state flickering during reconnect |
| bd-1u8 | closed | p2 | feature | Support vertical (portrait) orientation |
| bd-3aj | closed | p2 | feature | Zero-config USB detection and streaming |
| bd-2z7 | closed | p2 | feature | Bundle APK inside Mac app |
| bd-2do | closed | p2 | task | Ship binaries for all tagged releases (CI/CD) |
| bd-jyo | closed | p2 | bug | Fix corrupted APK in Homebrew cask |
| bd-3h2 | closed | p2 | task | Update README with honest alpha status |
| bd-13q | closed | p2 | feature | Add CLI interface for programmatic control |
| bd-1nn | closed | p3 | bug | Add Android app icon |
| bd-2d8 | closed | p3 | feature | Auto-reconnect on USB disconnect |
| bd-cb6 | closed | p3 | feature | Add menu bar connection status |

**Score: 18 closed, 3 open (1 epic, 1 task, 1 feature)**

---

### What Was Built/Fixed in Sprint 2

#### P1 Bug Fixes (3 closed)

1. **FPS fix (bd-1v0):** Root cause was the dirty-pixel window landing on `NSScreen.main` instead of the virtual display's NSScreen. Compositor now targets the correct display, with a 33ms timer fallback for mirror mode. Diagnostic logging confirms which path is active.

2. **Permissions fix (bd-ikz):** Three-layer permission defense -- UI gate, engine gate, capture gate. Cleanup on failure now tears down pacer, servers, and display manager properly. Clear error messages pointing to System Settings.

3. **Blank screen fix (bd-2fw):** Race condition where ADB tunnel + app launch happened before TCP server was listening. Startup reordered: servers listen first, then tunnel + launch. Tunnel result is now checked instead of silently discarded.

#### P2 Features & Fixes (7 closed)

4. **Connection flicker fix (bd-2x2):** 2s debounce (up from 500ms) covers the 1s reconnect cycle. Three-tier progressive status: silent for brief outages, "Reconnecting..." after 2s, full hints at 10s.

5. **Portrait orientation (bd-1u8):** Three portrait presets added -- Portrait Cozy (600x800 @2x = 1200x1600, matches DC-1 native panel exactly), Portrait Balanced (960x1280), Portrait Sharp (1200x1600 @1x). SwiftUI auto-discovers via `ForEach(DisplayResolution.allCases)`.

6. **Android app icon (bd-1nn):** Five PNG densities (48-192px) + vector drawable foreground. Minimal monitor outline, black-on-white. Adaptive icon config for Android 8+.

7. **CI/CD workflow (bd-2do):** GitHub Actions release pipeline with parallel Mac + Android builds. Version extraction from git tag. DMG creation with hdiutil fallback. Asset upload to GitHub Releases.

8. **Zero-config USB (bd-3aj):** `USBDeviceMonitor` polls `adb devices` every 2s on background queue. Edge-triggered callbacks. New `.waitingForDevice` status. Auto-start on connect, auto-stop on disconnect. Orange dot in menu bar during waiting state.

9. **Bundle APK (bd-2z7):** Mac app checks for bundled APK at `Contents/Resources/app-debug.apk`, auto-installs to DC-1 if companion app is missing. `make install` copies APK into bundle if available.

10. **Auto-reconnect (bd-2d8):** UI toggle for `autoMirrorEnabled` with UserDefaults persistence. Full disconnect/reconnect cycle: unplug -> waitingForDevice -> replug -> auto-resume.

#### P3 Polish (1 closed)

11. **Menu bar status (bd-cb6):** Colored dot next to display icon -- green (running), orange (transitional), red (error), none (idle). Extracted into compile-time-exhaustive helper function.

#### Infrastructure

12. **ADB embedding (bd-1dy, partially done):** `ADBBridge` rewritten to check `Bundle.main.resourcePath/adb` before PATH fallback. `make fetch-adb` downloads platform-tools. `make install` copies binary into bundle. Code is merged but bead remains open because no actual binary is shipped in git (by design) and notarization is unaddressed.

---

### What Remains and Why

| Bead | Why open | Blocker |
|------|----------|---------|
| bd-19o (epic) | Parent epic. Stays open until all child work is done. | bd-36n, bd-1dy still open |
| bd-36n (demo video) | Requires real DC-1 hardware test + recording. All code prerequisites are met. | Physical hardware access |
| bd-1dy (embed ADB) | Code is done, but the bead tracks the full story: unsigned Google binary needs notarization coverage when the app gets Developer ID signing. | Apple Developer signing |

---

### Issues Discovered During Sprint 2

1. **br dependency enforcement may be broken.** bd-1dy has a `blocks` relationship to bd-2z7, yet `br ready` returned bd-2z7 as ready. Either the dependency direction was stored inverted or br does not enforce `blocks` edges. bd-2z7 was completed successfully regardless, so this was not blocking in practice. Worth investigating in br itself.

2. **No hardware validation yet.** All fixes are code-reviewed and compile-clean, but zero real-hardware testing has occurred. The FPS fix (bd-1v0) specifically noted that mirror-mode timer fallback may be the active path on real hardware, which would need a different approach.

3. **CI/CD uses outdated action.** `softprops/action-gh-release@v1` should be upgraded to v2. Low urgency.

4. **APK version comparison missing.** Bundle APK install skips if package exists regardless of version. Future updates would require manual reinstall or a versionCode check.

5. **No UI surface for `apkInstallStatus`.** The published property exists in MirrorEngine but the SwiftUI view does not display it yet.

---

### Recommended Next Actions for Sprint 3

**Immediate (before anything else):**
1. **Test on real DC-1 hardware.** This is the single highest-leverage action. Every fix in Sprint 2 is unvalidated on metal. Connect, mirror, verify FPS, test reconnect, test portrait mode.

2. **Record demo video (bd-36n).** The hardware test IS the demo recording. Follow the script in demo_check.md. This proves the product works and unblocks external sharing.

**After hardware validation:**
3. **Apple Developer ID signing + notarization.** Required for distribution. Covers the embedded ADB binary (bd-1dy) and removes the quarantine/Gatekeeper friction that currently requires `xattr` workaround.

4. **First GitHub Release tag push.** Tests the CI/CD pipeline (bd-2do) end-to-end. Produces the first downloadable DMG + APK artifacts.

5. **Surface `apkInstallStatus` in SwiftUI.** Small UI task to show install progress in the menu bar dropdown.

6. **Upgrade `softprops/action-gh-release` to v2.** Trivial CI change.

**Stretch:**
7. **IOKit USB detection.** Once the DC-1's USB vendor/product ID is known from hardware testing, replace ADB polling with proper IOKit notifications for instant detection and lower overhead.

8. **APK version comparison.** Check installed versionCode against bundled APK to enable automatic updates.

---

### Changes Ready to PR

All Sprint 2 work is on `main` already (committed directly). The following are the logical change sets that could be tagged as a release:

| Change set | Files | Lines | Status |
|------------|-------|-------|--------|
| P1 bug fixes (FPS, permissions, blank screen) | 4 files | +182/-50 | Reviewed, no rework needed |
| P2 features (flicker, portrait, icon, CI/CD) | 8+ files | ~500 lines | Reviewed, no rework needed |
| Zero-config USB + auto-reconnect + menu status | 3 files | ~300 lines | Reviewed, no rework needed |
| ADB embed + APK bundle | 3 files | ~150 lines | Reviewed, code done, needs hardware test |

**Recommendation:** Tag `v1.4.0` after hardware validation confirms the fixes work on a real DC-1. All code is on main, build passes, reviews are clean. The tag push will trigger the CI/CD pipeline and produce the first automated release artifacts.

---

### Honest Assessment

**What worked:**
- Systematic triage by priority (P1 bugs first, then P2 features, then P3 polish) was effective. Every P1 had a clear root cause and a focused fix.
- Code reviews caught no rework-worthy issues across 11 changes. The implementation quality was high.
- The zero-config USB + auto-reconnect + bundle APK chain transforms the setup experience from "install 3 things manually" to "plug in and go."

**What didn't happen:**
- Zero hardware testing. Every claim about "30 FPS" and "works on first connect" is based on code analysis, not observation. The FPS fix explicitly noted a fallback path that may be active on real hardware.
- No release artifacts exist yet. The CI/CD pipeline has never been triggered.
- The epic (bd-19o) is still open because the demo video -- the one deliverable that proves the product works -- has not been recorded.

**Bottom line:** Sprint 2 shipped a large volume of well-reviewed code. Sprint 3 must validate it on hardware. Code without hardware testing is speculation.
