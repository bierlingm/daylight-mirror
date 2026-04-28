# codebase

kind: let
source: session: lead (Phase 1 exploration)

---

## Repo Structure
- Sources/MirrorEngine/MirrorEngine.swift (1,769 LOC) — core engine, all classes
- Sources/App/DaylightMirrorApp.swift — SwiftUI menu bar app
- Sources/Mirror/main.swift — CLI executable
- Sources/CLZ4/ — LZ4 C compression lib
- android/app/ — Kotlin + NDK companion app
- Package.swift — SPM (macOS 14+)
- Makefile — build orchestration

## Build System
- Mac: SPM (`swift build -c release`), installed to ~/Applications as .app bundle
- Android: Gradle + CMake, APK output: android/app/build/outputs/apk/debug/app-debug.apk

## Key Architecture
- ScreenCaptureKit (SCStream) captures virtual display → greyscale → LZ4 compress → TCP:8888
- Android receives frames, LZ4 decompress, NEON SIMD delta XOR, blit to ANativeWindow
- Unix domain socket IPC at /tmp/daylight-mirror.sock for CLI control
- 4 resolutions: cozy(800x600 HiDPI), comfortable(1024x768), balanced(1280x960), sharp(1600x1200)

## Code Signing
- Makefile creates bundle but signing is broken
- Gatekeeper rejects: "code has no resources but signature indicates they must be present"

## Android Package
- Actual: com.daylight.mirror (in build.gradle.kts and AndroidManifest.xml)
- Docs reference: com.welfvh.daylightmirror (mismatch)

## Beads State: 13 issues, 8 ready (bd-fma, bd-1hs, bd-39g, bd-1v0, bd-36n, bd-2do, bd-3h2, bd-19o)
