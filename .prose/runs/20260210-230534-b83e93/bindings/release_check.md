# release_check
kind: let
---

# Release Binary Shipping Analysis for v1.1-v1.3

## Current State

### GitHub Releases Status
- **v1.0**: 4 assets (app-debug.apk, 3x DMG variants) - fully shipped
- **v1.1**: 0 assets - **missing binaries**
- **v1.2**: 0 assets - **missing binaries**
- **v1.3**: 2 assets (app-debug.apk, DaylightMirror-v1.3.dmg) - partially shipped

### Missing CI Infrastructure
- No `.github/workflows/` directory exists
- All releases appear to be manual builds
- No automation for binary builds on tag push

## Findings

### 1. Build Process (from Makefile)

**Mac App:**
```bash
cd server && swift build -c release
# Produces: server/.build/release/DaylightMirror
```

**Android APK:**
```bash
cd android && ./gradlew assembleDebug
# Produces: android/app/build/outputs/apk/debug/app-debug.apk
```

**Mac App Bundle:**
```bash
make install
# Creates ~/Applications/Daylight Mirror.app
# Structure: .app/Contents/MacOS/DaylightMirror + Info.plist + Resources/AppIcon.icns
# Codesigns with ad-hoc signature
```

### 2. Version Management Issues

**Info.plist** is hardcoded to version 1.0:
```xml
<key>CFBundleShortVersionString</key>
<string>1.0</string>
<key>CFBundleVersion</key>
<string>1.0</string>
```

**Android build.gradle.kts** is also hardcoded:
```kotlin
versionCode = 1
versionName = "1.0"
```

These need to be updated for each release or parameterized.

### 3. DMG Creation

v1.0 and v1.3 both have DMG files, but there's no script in the repo for creating them. This is likely done manually using:
- `hdiutil` commands
- Or `create-dmg` tool (not in repo)
- Or macOS Disk Utility

### 4. What's Needed for v1.1 and v1.2

To retroactively ship v1.1 and v1.2:

1. **Checkout each tag**
2. **Build Mac binary**: `cd server && swift build -c release`
3. **Build Android APK**: `cd android && ./gradlew assembleDebug`
4. **Create .app bundle** (manual or script)
5. **Create DMG** (need script or manual process)
6. **Upload to GitHub release**: `gh release upload v1.1 DaylightMirror-v1.1.dmg app-debug.apk`

## Recommended Solution: GitHub Actions Workflow

### Proposed Workflow

Create `.github/workflows/release.yml`:

```yaml
name: Release Binaries

on:
  push:
    tags:
      - 'v*'

jobs:
  build-mac:
    runs-on: macos-latest

    steps:
      - uses: actions/checkout@v4

      - name: Extract version from tag
        id: version
        run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

      - name: Build Mac binary
        run: |
          cd server
          swift build -c release

      - name: Create .app bundle
        run: |
          APP_NAME="Daylight Mirror"
          APP_BUNDLE="build/$APP_NAME.app"
          BINARY="server/.build/release/DaylightMirror"

          mkdir -p "$APP_BUNDLE/Contents/MacOS"
          mkdir -p "$APP_BUNDLE/Contents/Resources"
          cp "$BINARY" "$APP_BUNDLE/Contents/MacOS/DaylightMirror"

          # Update Info.plist with version
          sed "s/1.0/${{ steps.version.outputs.VERSION }}/g" Info.plist > "$APP_BUNDLE/Contents/Info.plist"

          cp Resources/AppIcon.icns "$APP_BUNDLE/Contents/Resources/AppIcon.icns"
          codesign --force --deep -s - "$APP_BUNDLE"

      - name: Create DMG
        run: |
          brew install create-dmg
          create-dmg \
            --volname "Daylight Mirror ${{ steps.version.outputs.VERSION }}" \
            --window-pos 200 120 \
            --window-size 600 400 \
            --icon-size 100 \
            --icon "Daylight Mirror.app" 175 120 \
            --hide-extension "Daylight Mirror.app" \
            --app-drop-link 425 120 \
            "DaylightMirror-v${{ steps.version.outputs.VERSION }}.dmg" \
            "build/Daylight Mirror.app"

      - name: Upload DMG
        uses: actions/upload-artifact@v4
        with:
          name: mac-dmg
          path: DaylightMirror-v${{ steps.version.outputs.VERSION }}.dmg

  build-android:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Setup Android SDK
        uses: android-actions/setup-android@v3

      - name: Extract version from tag
        id: version
        run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

      - name: Update version in build.gradle.kts
        run: |
          cd android/app
          # Update versionName
          sed -i "s/versionName = \"1.0\"/versionName = \"${{ steps.version.outputs.VERSION }}\"/" build.gradle.kts

      - name: Build APK
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleDebug

      - name: Rename APK
        run: |
          cp android/app/build/outputs/apk/debug/app-debug.apk \
             DaylightMirror-v${{ steps.version.outputs.VERSION }}.apk

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: android-apk
          path: DaylightMirror-v${{ steps.version.outputs.VERSION }}.apk

  create-release:
    needs: [build-mac, build-android]
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - uses: actions/checkout@v4

      - name: Extract version from tag
        id: version
        run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

      - name: Download Mac artifact
        uses: actions/download-artifact@v4
        with:
          name: mac-dmg

      - name: Download Android artifact
        uses: actions/download-artifact@v4
        with:
          name: android-apk

      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            DaylightMirror-v${{ steps.version.outputs.VERSION }}.dmg
            DaylightMirror-v${{ steps.version.outputs.VERSION }}.apk
          draft: false
          prerelease: false
          generate_release_notes: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Key Features

1. **Triggered on tag push**: Any `v*` tag triggers the workflow
2. **Version extraction**: Automatically pulls version from tag name
3. **Parallel builds**: Mac and Android build simultaneously
4. **Version patching**: Updates Info.plist and build.gradle.kts with correct version
5. **DMG creation**: Uses `create-dmg` for professional-looking installers
6. **Auto-release**: Creates GitHub release with both binaries attached

## Manual Build Steps (for retroactive v1.1/v1.2 release)

If you want to manually build and upload v1.1 and v1.2 right now:

```bash
#!/bin/bash
# Build script for specific tag

TAG="v1.1"  # Change to v1.2 for second build
VERSION="${TAG#v}"

# Checkout tag
git checkout $TAG

# Build Mac binary
cd server
swift build -c release
cd ..

# Create .app bundle
APP_BUNDLE="build/Daylight Mirror.app"
mkdir -p "$APP_BUNDLE/Contents/MacOS"
mkdir -p "$APP_BUNDLE/Contents/Resources"
cp server/.build/release/DaylightMirror "$APP_BUNDLE/Contents/MacOS/DaylightMirror"
sed "s/1.0/$VERSION/g" Info.plist > "$APP_BUNDLE/Contents/Info.plist"
cp Resources/AppIcon.icns "$APP_BUNDLE/Contents/Resources/AppIcon.icns"
codesign --force --deep -s - "$APP_BUNDLE"

# Create DMG (requires create-dmg: brew install create-dmg)
create-dmg \
  --volname "Daylight Mirror $VERSION" \
  --window-pos 200 120 \
  --window-size 600 400 \
  --icon-size 100 \
  --icon "Daylight Mirror.app" 175 120 \
  --hide-extension "Daylight Mirror.app" \
  --app-drop-link 425 120 \
  "DaylightMirror-$TAG.dmg" \
  "$APP_BUNDLE"

# Build Android APK (requires Android SDK)
cd android
./gradlew assembleDebug
cd ..
cp android/app/build/outputs/apk/debug/app-debug.apk "DaylightMirror-$TAG.apk"

# Upload to GitHub release
gh release upload $TAG "DaylightMirror-$TAG.dmg" "DaylightMirror-$TAG.apk"

# Return to main branch
git checkout main
```

## Recommendations

### Immediate Actions

1. **Add GitHub Actions workflow** (draft provided above)
2. **Test workflow** by creating a test tag (v1.3.1-test) and verifying it builds correctly
3. **Manually build v1.1 and v1.2** using the script above
4. **Update Homebrew Cask** formula if needed after releases

### Long-term Improvements

1. **Version automation**: Create a script to bump version numbers in Info.plist and build.gradle.kts
2. **Signed binaries**: Set up proper code signing for Mac (requires Apple Developer account)
3. **Release APK**: Consider building release APK instead of debug (requires keystore)
4. **Notarization**: Add macOS notarization for better security (requires Apple Developer account)
5. **Checksums**: Add SHA256 checksums to release notes

### Blocker Resolution

For bd-2do, the path forward is:

1. Create the GitHub Actions workflow (no push needed, just have it ready)
2. Document that v1.1 and v1.2 need manual builds (or run the script)
3. Verify v1.3 release is complete (it has both assets already)
4. Future releases will be automated once workflow is merged

## Files That Need Creation

- `.github/workflows/release.yml` - GitHub Actions workflow
- `scripts/build-release.sh` - Manual release build script (optional, for ad-hoc builds)
- `scripts/bump-version.sh` - Version bumping automation (optional)
