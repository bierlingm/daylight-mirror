# zero_config

kind: let

---

## Summary

Implemented zero-config USB detection using ADB device polling. The app now automatically detects when a Daylight DC-1 is connected/disconnected via USB and starts/stops mirroring without user intervention.

Since the DC-1's USB vendor/product ID is unknown, the implementation polls `adb devices` every 2 seconds on a background thread rather than using IOKit USB notifications.

## Changes

### `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/MirrorEngine/MirrorEngine.swift`

**New `MirrorStatus.waitingForDevice` case**: Added between `.idle` and `.starting`. Shown when the mirror was auto-stopped due to USB disconnect and is waiting for the device to reconnect.

**New `USBDeviceMonitor` class** (before MirrorEngine):
- Polls `ADBBridge.connectedDevice()` every 2 seconds on a utility-QoS background queue
- Tracks connect/disconnect transitions (edge-triggered, not level)
- Fires `onDeviceConnected` / `onDeviceDisconnected` callbacks on the main queue
- Gracefully disables itself if no adb binary is available

**New MirrorEngine properties**:
- `@Published deviceDetected: Bool` -- reflects real-time USB device state
- `autoMirrorEnabled: Bool` -- controls whether auto-start/stop is active (default: true)
- `usbMonitor: USBDeviceMonitor?` -- private, started in init

**Auto-mirror behavior** (in MirrorEngine.init):
- On device connect + idle: auto-starts mirroring
- On device connect + waitingForDevice: resumes mirroring
- On device connect + running: re-establishes ADB tunnel (reconnect)
- On device disconnect + running: stops mirroring, sets status to `.waitingForDevice`

**Updated methods**:
- `stop()`: handles `.waitingForDevice` state (resets to idle without teardown)
- `toggleMirror()`: accepts `.waitingForDevice` as a startable state
- Control socket `STATUS` command: returns `waiting_for_device` for the new state

### `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/App/DaylightMirrorApp.swift`

**Menu bar icon**: Shows orange dot when in `.waitingForDevice` state.

**Status badge**: Maps `.waitingForDevice` to orange color and "Waiting for DC-1" text.

**New `waitingForDeviceView`**: Shows cable icon with "Connect your Daylight" message and a Cancel button.

**Idle view**: Now shows device detection status with a green/gray dot ("DC-1 detected via USB" / "No device connected").

**Switch exhaustiveness**: All `switch engine.status` blocks handle the new case.

### `/Users/moritzbierling/werk/wield/daylight-mirror/Sources/Mirror/main.swift`

Added `waitingForDevice` case to the CLI status polling switch.

## Build verification

`swift build -c release` passes cleanly (2.89s).

## Design decisions

1. **ADB polling over IOKit**: Without the DC-1's USB vendor/product ID, IOKit filtering would match all USB devices. Polling `adb devices` every 2s is simple, correct, and costs ~2ms per poll.

2. **Edge-triggered callbacks**: The monitor only fires on transitions (connected->disconnected, disconnected->connected), not on every poll tick. This prevents duplicate start/stop calls.

3. **`waitingForDevice` state**: Distinguishes "user clicked Stop" (-> idle) from "device was unplugged" (-> waitingForDevice). In the latter case, plugging the device back in resumes mirroring automatically.

4. **`autoMirrorEnabled` flag**: Allows disabling auto-mirror behavior if users prefer manual control. Defaults to true for zero-config experience.
