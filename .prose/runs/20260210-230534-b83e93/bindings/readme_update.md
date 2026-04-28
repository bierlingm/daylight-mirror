# readme_update
kind: let
---
README updated with honest alpha status and known limitations.

## Changes Made

1. **Alpha badge** added prominently at the top with positive framing: "actively maintained, community-tested, stable enough for daily use"

2. **Performance claims revised** — replaced "30 FPS. Under 10ms latency." with honest current behavior:
   - "Low-latency. Lossless. Zero artifacts."
   - Frame rate varies with screen activity (~13 FPS static, higher with movement)
   - Explained this is due to macOS compositor optimization

3. **New "Known Issues & Limitations" section** added before CLI:
   - FPS variance with screen activity (compositor optimization, not a bug)
   - USB-C only (no WiFi streaming)
   - Requires macOS 14+ and Daylight DC-1 specifically
   - Community validation note (Moritz Bierling confirmed working)

4. **Removed xattr workaround** — not listed in limitations since code signing was fixed

## Voice

The tone is honest but not apologetic. It frames alpha status as "actively maintained" and "stable enough for daily use" — truthful about limitations while emphasizing the project is a serious, maintained effort. The FPS explanation is technical and factual, not defensive.

## What Was NOT Changed

- Setup instructions (accurate)
- Feature descriptions (accurate)
- Blog series links (accurate)
- License and support sections (accurate)

The README now reflects the actual current state of the software without underselling or overselling it.
