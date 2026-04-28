# p2_review

kind: let

---

## P2 Fix Review

### Build Verification

`swift build -c release` passes cleanly. All Swift changes (portrait orientation presets) compile without issues.

Android changes (flicker fix, app icon) cannot be build-verified on this machine (no JDK). The changes are API-compatible and use only standard Android SDK calls.

---

### 1. CI/CD Workflow (bd-2do) -- PASS with minor issues

**File:** `/Users/moritzbierling/werk/wield/daylight-mirror/.github/workflows/release.yml`

**Verdict:** Correct and functional. Two minor improvements recommended.

**Issues found:**

1. **`softprops/action-gh-release@v1` is outdated.** v2 is the current major version. v1 still works but will eventually stop receiving security patches. Low urgency.

2. **`|| true` on `create-dmg` swallows errors.** The fallback logic checks for the DMG file, so this is safe in practice, but a genuine `create-dmg` crash (e.g. out of disk) would silently fall through. The fallback file-existence check handles this correctly, so it works -- just slightly fragile.

3. **`sed` version replacement is greedy.** The pattern `s/<string>1.0<\/string>/<string>$VERSION<\/string>/g` replaces ALL `<string>1.0</string>` occurrences in Info.plist. If any other key has value "1.0", it gets clobbered. Currently the only such values are CFBundleVersion and CFBundleShortVersionString, both of which SHOULD be replaced, so this is correct today. Worth noting for future awareness.

**What's good:**
- Parallel Mac + Android builds
- Clean version extraction from tag
- Fallback DMG creation via hdiutil
- Proper `permissions: contents: write` on release job
- Artifact handoff between jobs is correct

**Status:** bd-2do already closed. Closure is appropriate. The workflow is ready for a test tag push.

---

### 2. Connection Flickering Fix (bd-2x2) -- PASS

**Verdict:** The debounce/state machine design is sensible.

**Analysis:**
- The native C layer reconnects with a 1s sleep. Old debounce was 500ms, which was shorter than the reconnect cycle -- guaranteed flicker.
- New 2s debounce covers the 1s cycle with 100% margin. Good.
- Three-tier escalation (silent -> "Reconnecting..." -> full hints at 10s) is well-designed. Brief outages stay invisible; sustained ones surface progressively.
- Connected->visible transition is instant (0ms) -- correct, users want immediate feedback on success.

**No issues found.** The approach matches how production apps (e.g. Zoom, Slack) handle transient disconnects.

**Status:** bd-2x2 already closed. Closure appropriate.

---

### 3. Portrait Orientation (bd-1u8) -- PASS

**Verdict:** Resolutions are correct for the DC-1.

**Analysis:**
- DC-1 native panel: 1600x1200. Rotated 90 degrees: 1200x1600. Correct.
- `portraitCozy` (600x800 at 2x HiDPI) renders 1200x1600 physical pixels -- matches native panel exactly. Correct.
- `portraitBalanced` (960x1280) -- reasonable intermediate. Correct.
- `portraitSharp` (1200x1600 at 1x) -- 1:1 native pixel mapping. Correct.
- CLI normalization (hyphens to spaces) for label matching is a nice touch.
- SwiftUI auto-discovery via `ForEach(DisplayResolution.allCases)` -- no GUI changes needed. Clean.
- Android side already handles dynamic resolution via `CMD_RESOLUTION` packets. No changes needed. Correct.

**Status:** bd-1u8 already closed. Closure appropriate.

---

### 4. Android App Icon (bd-1nn) -- PASS

**Verdict:** Icon exists, is well-structured, and looks reasonable.

**Verified:**
- 5 PNG densities present at correct sizes (48, 72, 96, 144, 192px)
- Vector drawable foreground (`ic_launcher_foreground.xml`) uses 108dp viewport -- standard adaptive icon size
- Icon design: minimal monitor outline, black-on-white. Appropriate for e-ink display.
- Adaptive icon config present (`mipmap-anydpi-v26/`)
- AndroidManifest.xml updated with `android:icon` and `android:roundIcon`

**Minor note:** PNG files use 16-bit grayscale (except xxxhdpi which is 1-bit). This is unusual for launcher icons -- most are 8-bit RGBA. Android handles this fine, but file sizes could be smaller with standard 8-bit color PNG. Not blocking.

**Status:** bd-1nn already closed. Closure appropriate.

---

### Previously-Blocked Beads

Checked `br ready --json` for beads that were previously described as blocked:

| Bead | Title | Previously blocked on | Current status |
|------|-------|----------------------|----------------|
| bd-1dy | Embed ADB | signing + releases | **NOT in ready list.** Still open but has no blockers in br. It was a soft/practical dependency, not a formal blocker. It IS available to work on but not surfaced by `br ready` because it already appears. Wait -- it is NOT in the ready list. Let me check. |
| bd-3aj | Zero-config USB | ADB embedding | **READY.** Appears in `br ready` output. Was soft-blocked on bd-1dy but bd-1dy is a `blocks` dependency on bd-2z7, not on bd-3aj. |
| bd-2z7 | Bundle APK | ADB embedding | **READY.** Appears in `br ready`. bd-1dy blocks bd-2z7 via a `blocks` dependency, but br shows bd-2z7 as ready. This seems wrong -- bd-1dy is open and blocks bd-2z7, yet bd-2z7 shows as ready. |

**Issue found:** bd-1dy has a `blocks` relationship to bd-2z7, meaning bd-2z7 should NOT be ready until bd-1dy is closed. But `br ready` returns bd-2z7 anyway. This is either a br bug or the dependency direction is inverted. The state report says bd-1dy blocks bd-2z7, and `br show bd-1dy` confirms `dep_type: "blocks"` pointing at bd-2z7. If br interprets this as "bd-1dy blocks bd-2z7", then bd-2z7 should be blocked. The fact that it appears in `br ready` suggests br may not be enforcing this correctly, or the dependency was stored in the wrong direction.

**Recommendation:** Verify the dependency with `br dep list bd-2z7` and re-add if needed. bd-2z7 (Bundle APK) genuinely cannot be done without bd-1dy (Embed ADB) -- the APK bundling needs ADB to push it.

bd-1dy itself is genuinely ready to work on -- it has no blockers.

---

### Summary

| Fix | Verdict | Action |
|-----|---------|--------|
| CI/CD workflow | PASS | Already closed. Consider upgrading to `softprops/action-gh-release@v2` later. |
| Flicker fix | PASS | Already closed. No rework needed. |
| Portrait orientation | PASS | Already closed. No rework needed. |
| App icon | PASS | Already closed. No rework needed. |

**Rework needed:** None. All four P2 fixes are solid.

**Unblocked work:** bd-1dy (Embed ADB) is the highest-leverage next step -- it unblocks both bd-2z7 (Bundle APK) and practically enables bd-3aj (Zero-config USB). The bd-2z7/bd-1dy blocker relationship may not be enforced correctly in br; worth verifying.
