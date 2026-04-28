# demo_check

kind: let

---

## Demo Video Feasibility (bd-36n)

**Verdict: FEASIBLE -- all blockers are resolved.**

All three P1 bugs that constituted the soft prerequisites for a working end-to-end flow are closed and code-reviewed:

| Bug | Fix | Status |
|-----|-----|--------|
| bd-1v0 (FPS) | Dirty-pixel window moved to virtual display's NSScreen; timer fallback for mirror mode | Closed, reviewed, no rework |
| bd-ikz (Permissions) | Three-layer permission gate + proper cleanup on failure | Closed, reviewed, no rework |
| bd-2fw (Blank screen) | Startup reordered so servers listen before ADB tunnel; tunnel result checked | Closed, reviewed, no rework |

Build passes cleanly. P2 fixes (flicker debounce, portrait presets, app icon, CI/CD) are also closed and reviewed.

**One caveat:** None of this has been tested on real hardware with a DC-1 yet. The code review confirms correctness, but the demo recording IS the hardware test. Be prepared for one iteration if the mirror-mode timer fallback is the active path (bd-1v0 noted this possibility).

---

## Recommended Demo Script

The demo should show the full first-run experience in under 90 seconds:

1. **Install** -- Open DMG, drag to Applications, launch from menu bar
2. **Permissions** -- Show the setup wizard prompting for Screen Recording permission; grant it
3. **Connect** -- Plug DC-1 via USB, show the app detecting the device
4. **Mirror** -- Click "Start Mirror", show macOS screen appearing on DC-1 at ~30 FPS
5. **Resolution switching** -- Cycle through at least two presets (e.g. Cozy HiDPI -> Balanced) to show the resolution picker working
6. **Portrait mode** -- Switch to a portrait preset, show the DC-1 updating orientation
7. **Stats overlay** -- Briefly show the FPS/latency stats to confirm performance
8. **Disconnect/reconnect** -- Unplug and replug USB to show the flicker-free reconnection (2s debounce, progressive status)

**Nice to have (if time permits):**
- CLI usage: `daylight-mirror --resolution "Cozy HiDPI"` from terminal
- Sharpening slider adjustment

**Recording notes:**
- Use QuickTime or OBS for the Mac side
- Point a phone camera at the DC-1 to capture its e-ink display (screen recording won't capture it)
- Split-screen or picture-in-picture showing Mac UI + DC-1 physical output

---

## Bead Update

bd-36n status remains `open` with no formal blockers. It is ready to execute. Recommended next step: test on real hardware, then record.
