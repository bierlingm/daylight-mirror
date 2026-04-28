# auto_reconnect

kind: let

---

## Summary

Auto-reconnect on USB disconnect was already implemented by bd-3aj (zero-config USB detection). The `USBDeviceMonitor` class polls `adb devices` every 2 seconds and the `MirrorEngine.init` wires up connect/disconnect callbacks that handle all the required behavior. The missing piece was a **UI toggle** for the `autoMirrorEnabled` property and **persistence** of that preference across launches.

## What was already in place (from bd-3aj)

1. **On disconnect**: `USBDeviceMonitor.onDeviceDisconnected` fires, engine stops streaming, status becomes `.waitingForDevice`, menu bar shows orange dot with "Waiting for DC-1"
2. **On reconnect**: `USBDeviceMonitor.onDeviceConnected` fires, engine either auto-starts (from idle/waitingForDevice) or calls `reconnect()` (from running) which re-establishes the ADB tunnel and relaunches the Android app
3. **Polling**: Every 2 seconds on a utility-QoS background queue, edge-triggered (only fires on transitions)

## Changes made

### `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/MirrorEngine/MirrorEngine.swift`

**`autoMirrorEnabled` made `@Published` with UserDefaults persistence**:
- Changed from plain `Bool` to `@Published` with a `didSet` that saves to `UserDefaults` under key `"autoMirrorEnabled"`
- In `init()`, reads the saved value if one exists (defaults to `true` for zero-config experience)
- This allows the SwiftUI toggle in the menu bar to bind directly via `$engine.autoMirrorEnabled`

### `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/App/DaylightMirrorApp.swift`

**Added auto-reconnect toggle to the running view**: A `Toggle` with switch style bound to `$engine.autoMirrorEnabled`, placed between the sharpen slider section and the reconnect/restart/stop buttons. Label: "Auto-reconnect on USB" with a cable.connector icon.

## Build verification

`swift build -c release` passes cleanly (6.95s).

## How it works end-to-end

1. User unplugs USB cable
2. `USBDeviceMonitor` detects no device on next 2s poll
3. `onDeviceDisconnected` fires on main queue
4. If `autoMirrorEnabled`: engine stops, status -> `.waitingForDevice`, menu bar shows orange dot
5. User plugs cable back in
6. `USBDeviceMonitor` detects device on next 2s poll
7. `onDeviceConnected` fires on main queue
8. If `autoMirrorEnabled` and status is `.waitingForDevice`: engine auto-starts, re-establishes ADB tunnel, relaunches Android app
9. Menu bar returns to green dot

The toggle in the menu lets users disable this behavior if they prefer manual control. The preference persists across app launches.
