# signing_fix
kind: let
---

## Root cause

The `make install` target had three problems that caused the 'code has no resources but signature indicates they must be present' Gatekeeper error:

1. **Info.plist created inline via PlistBuddy** instead of copying the repo's `Info.plist`. The repo version declares `CFBundleIconFile = AppIcon`, but PlistBuddy never added that key -- so the bundle had no icon reference and no resource files at all.

2. **Resources directory never created; icon never copied.** `Resources/AppIcon.icns` exists in the repo but was never placed into `Contents/Resources/` inside the `.app` bundle.

3. **No codesign step.** The Swift compiler signs the bare binary during `swift build`, but once that binary is copied into a hand-assembled `.app` bundle the original signature becomes invalid (the bundle structure doesn't match). Without a fresh ad-hoc signature on the whole bundle, Gatekeeper rejects it.

## Fix applied

File: `/Users/moritzbierling/werk/wield/daylight-mirror/Makefile`, `install` target.

Replaced the PlistBuddy block with three lines:

```makefile
@mkdir -p "$(APP_BUNDLE)/Contents/Resources"
@cp Info.plist "$(APP_BUNDLE)/Contents/Info.plist"
@cp Resources/AppIcon.icns "$(APP_BUNDLE)/Contents/Resources/AppIcon.icns"
@codesign --force --deep -s - "$(APP_BUNDLE)"
```

- Copies the canonical `Info.plist` (single source of truth, includes icon key).
- Copies the `.icns` icon into the bundle's Resources directory.
- Runs `codesign --force --deep -s -` to apply a valid ad-hoc signature to the entire bundle, which satisfies Gatekeeper.

## Verification

After `make install`, run:

```bash
codesign --verify --verbose=2 ~/Applications/Daylight\ Mirror.app
```

Expected output: `valid on disk` with no errors.
