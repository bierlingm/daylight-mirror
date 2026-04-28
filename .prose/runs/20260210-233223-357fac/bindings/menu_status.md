# menu_status

kind: let

---

## Summary

Added connection status indicator to the menu bar icon. A colored dot appears next to the display icon reflecting the current engine state:

- **Green dot**: connected and streaming (`.running`)
- **Orange dot**: reconnecting or transitioning (`.waitingForDevice`, `.starting`, `.stopping`)
- **Red dot**: error (`.error`)
- **No dot**: idle/stopped (`.idle`)

## Changes

### `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/App/DaylightMirrorApp.swift`

**Refactored menu bar label** (lines 74-83): Replaced inline conditional logic with a call to `menuBarDotColor(_:)`, which maps all six `MirrorStatus` cases to an optional `Color`. When the function returns `nil` (idle), no dot is rendered.

**New `menuBarDotColor(_:)` method** on `DaylightMirrorApp`: A single exhaustive switch over `MirrorStatus` returning `.green`, `.orange`, `.red`, or `nil`. This centralizes the color mapping and ensures new status cases produce a compiler error if unhandled.

The existing `statusBadge` inside `MirrorMenuView` (the dropdown panel) was already fully mapped and required no changes.

## Build verification

`swift build` passes cleanly (4.21s).

## Design decisions

1. **Orange for transitional states**: `.starting` and `.stopping` are brief transitional states. Orange (same as `.waitingForDevice`) signals "something is happening" without introducing a fourth color.

2. **No dot for idle**: A missing dot is the cleanest idle indicator -- it avoids visual noise when the app is not active.

3. **Helper function over inline switch**: Extracting `menuBarDotColor` keeps the `body` declarative and makes the color mapping testable and exhaustive at compile time.
