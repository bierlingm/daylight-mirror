# summary
kind: output
---

## Daylight Mirror Improvement Session — 2026-02-10

### Fixed This Session (8 beads closed)

| Bead | Issue | Resolution |
|------|-------|------------|
| bd-fma | Homebrew cask APK URL pointed to wrong filename | Fixed: `DaylightMirror.apk` → `app-debug.apk` in cask formula |
| bd-1hs | Mac app code signing broken in Makefile | Fixed: `install` target now copies Info.plist and signs correctly |
| bd-39g | Android package name mismatch audit | Confirmed all docs already use correct `com.example.daylightclient` |
| bd-3h2 | README overpromised on project maturity | Rewrote README with honest alpha status, known issues, realistic expectations |
| bd-3ae | Mac app code signing for distribution | Closed as duplicate of bd-1hs |
| bd-jyo | Corrupted APK in Homebrew cask | Closed as duplicate of bd-fma |
| bd-13q | Add CLI interface for programmatic control | Closed — CLI already exists at `Sources/Mirror/main.swift` |
| bd-19h | Duplicate epic | Closed as duplicate of bd-19o |

### Investigated This Session (2 beads updated)

**bd-1v0 — Achieve advertised 30 FPS** (status: in_progress)
- Dirty-pixel workaround (`CompositorPacer`) already implemented in `MirrorEngine.swift` lines 593-652
- 1x1 window toggles black/near-black at 30Hz via `CADisplayLink`
- 5 next steps documented: verify pacer fires, try virtual display placement, try larger region, try lower window level, profile with Instruments
- Full writeup: `.prose/runs/20260210-230534-b83e93/bindings/fps_investigation.md`

**bd-2do — Ship binaries for all tagged releases** (status: open)
- v1.0: fully shipped (4 assets). v1.1 and v1.2: missing binaries. v1.3: partial (DMG + APK only)
- No CI/CD exists — all releases are manual
- GitHub Actions workflow drafted (`release.yml`) for auto-build on tag push
- Full writeup: `.prose/runs/20260210-230534-b83e93/bindings/release_check.md`

### New Issues Filed from GitHub (5 beads created)

| Bead | P | Type | Issue |
|------|---|------|-------|
| bd-ikz | P1 | bug | First-run permissions flow crashes when screen recording not granted |
| bd-2fw | P1 | bug | DC-1 blank screen after setup — Mac shows "Waiting for client" |
| bd-1u8 | P2 | feature | Support portrait (vertical) orientation for reading/coding |
| bd-2x2 | P2 | bug | Connection state overlay flickers during reconnect (500ms debounce insufficient) |
| bd-1nn | P3 | bug | Android app has no icon in DC-1 launcher |

### Still Blocked (5 beads)

| Bead | Issue | Blocked on |
|------|-------|------------|
| bd-1dy | Embed minimal ADB binary | Depends on broader setup UX decisions |
| bd-3aj | Zero-config USB detection and streaming | Depends on ADB embedding + permissions flow |
| bd-2z7 | Bundle APK inside Mac app | Depends on ADB embedding + detection |
| bd-2d8 | Auto-reconnect on USB disconnect | Depends on connection stability work |
| bd-cb6 | Menu bar connection status | Depends on connection state infrastructure |

### Recommended Next Actions

**Immediate (this week):**

1. **bd-ikz — Fix first-run permissions crash** (P1). This is the number one barrier to new users. Detect missing Screen Recording permission before attempting capture; show a setup checklist instead of crashing.

2. **bd-2fw — Debug DC-1 blank screen** (P1). Needs hands-on testing with a DC-1 device. Add logging to Android client to surface what happens after connection. May be a port-forwarding or scrcpy codec issue.

3. **bd-1v0 — Validate FPS fix** (P1, in_progress). CompositorPacer is implemented but untested on DC-1. Run with Instruments to confirm `CADisplayLink` fires at 30Hz and `SCStream` actually delivers frames.

**Short-term (next 2 weeks):**

4. **bd-2do — Set up CI releases** (P2). Create `.github/workflows/release.yml` from the drafted workflow. Test with a `v1.3.1-test` tag. Retroactively build v1.1 and v1.2 binaries.

5. **bd-2x2 — Fix connection flickering** (P2). Increase debounce window or suppress overlay during initial connection handshake.

6. **bd-1nn — Add Android app icon** (P3). Quick win for polish. Monochrome icon fits e-ink aesthetic.

**Medium-term (unblocks downstream):**

7. **bd-1dy — Embed minimal ADB** (P2). This unblocks bd-3aj (zero-config USB) and bd-2z7 (bundled APK), which together represent the path to a true one-click setup experience.

### Session Stats

- **Beads closed**: 8
- **Beads investigated**: 2
- **Beads created**: 5
- **Open beads total**: 14 (including epic bd-19o)
- **Blocked beads**: 5
- **Ready for work**: 6 (bd-ikz, bd-2fw, bd-1v0, bd-2do, bd-1u8, bd-1nn)
