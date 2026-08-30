# VinceBox iOS TestFlight Maintenance

This document records how this fork is maintained for VinceBox iOS/TestFlight builds.

## Repository Strategy

The fork should track upstream `dev`:

```bash
git fetch upstream dev
git rebase upstream/dev
```

Use `origin` for Vince's fork:

```bash
git remote -v
git push origin dev
```

Recommended local branch config:

```bash
git branch --set-upstream-to=upstream/dev dev
git config branch.dev.rebase true
git config branch.dev.pushRemote origin
```

This makes `git pull --rebase` follow `upstream/dev`, while normal pushes can still go to Vince's fork.

## Rebase Policy

Keep local commits narrow. The recurring local patch should mostly contain:

- Apple Team ID and bundle identifier changes.
- Entitlement values such as App Groups and iCloud containers.
- TestFlight/build signing adjustments.
- Local documentation.

Avoid changing source code unless there is a real feature or bug fix request. This reduces conflicts when rebasing onto upstream `dev`.

The highest-risk file is:

```text
sing-box.xcodeproj/project.pbxproj
```

During a rebase conflict, prefer keeping upstream's project structure first, then reapply Vince-specific values:

```text
io.vince.sfavt.app
group.io.vince.sfavt
iCloud.io.vince.sfavt
```

This matters because upstream `dev` may add new targets, build phases, or package references.

## Vince Identifiers

Apple Team ID is configured in the Xcode project build settings. Do not record personal account identifiers in this document.

Bundle identifiers:

```text
io.vince.sfavt.app
io.vince.sfavt.app.extension
io.vince.sfavt.app.intents
io.vince.sfavt.standalone
io.vince.sfavt.system
io.vince.sfavt.app.widget
io.vince.sfavt.app.fileprovider
io.vince.sfavt.app.share
io.vince.sfavt.app.action
```

Shared containers:

```text
group.io.vince.sfavt
iCloud.io.vince.sfavt
```

The iOS targets also need:

```text
BASE_PACKAGE_IDENTIFIER = io.vince.sfavt.app
APP_GROUP_IDENTIFIER = group.io.vince.sfavt
ICLOUD_CONTAINER_IDENTIFIER = iCloud.io.vince.sfavt
```

`BasePackageIdentifier` is a runtime input, not just a label. It drives `AppConfiguration.extensionBundleID`, `widgetControlKind`, `fileProviderDomainID`, `profileUTType`, and `backgroundTaskID`. If the main app Bundle ID changes, update `BASE_PACKAGE_IDENTIFIER` for `SFI`, `Extension`, `IntentsExtension`, `WidgetExtension`, `FileProviderExtension`, `ShareExtension`, and `ActionExtension` at the same time.

`SFI/Info.plist` keeps the iCloud ubiquitous container key as a literal:

```text
iCloud.io.vince.sfavt
```

Do not change it to `iCloud.$(BASE_PACKAGE_IDENTIFIER)` while the old iCloud container is intentionally reused.

## Multicast Networking

The iOS packet tunnel extension uses:

```text
com.apple.developer.networking.multicast
```

Apple has approved this entitlement for VinceBox. Within the iOS app targets, it is enabled in `Extension/Extension.entitlements` for the packet tunnel App ID `io.vince.sfavt.app.extension`.

The signed iOS archive produced on 2026-08-30 includes `com.apple.developer.networking.multicast` in the packet tunnel extension. Continue to verify this after refreshing its provisioning profile, and do not add it to the main app or unrelated extensions.

## Libbox

`Libbox.xcframework` is required but ignored by Git. If it is missing, build it from the matching `SagerNet/sing-box` version.

For upstream `dev` version `1.14.0-rc.5`, build libbox from:

```text
SagerNet/sing-box tag v1.14.0-rc.5
```

The local `Libbox.xcframework` for the dev branch was rebuilt on 2026-08-30 and copied to:

```text
<repo-root>/Libbox.xcframework
```

It contained:

```text
ios-arm64
ios-arm64_x86_64-simulator
tvos-arm64
tvos-arm64_x86_64-simulator
macos-arm64_x86_64
```

Using an older libbox with upstream `dev` causes Swift compile errors for missing APIs. For example, `v1.14.0-alpha.27` lacks the remote command, bridge, taildrop, and setup-option symbols required by `1.14.0-rc.5`.

## Verified Dev Build Command

From:

```text
<repo-root>
```

Run:

```bash
xcodebuild build \
  -project sing-box.xcodeproj \
  -scheme SFI \
  -configuration Release \
  -destination 'generic/platform=iOS' \
  -skipPackagePluginValidation \
  -skipMacroValidation \
  CODE_SIGNING_ALLOWED=NO
```

On 2026-05-31, after rebuilding `Libbox.xcframework` from `v1.14.0-alpha.27`, this completed with:

```text
** BUILD SUCCEEDED **
```

## Verified macOS Build Command

From:

```text
<repo-root>
```

Run:

```bash
xcodebuild build \
  -project sing-box.xcodeproj \
  -scheme SFM \
  -configuration Release \
  -destination 'generic/platform=macOS' \
  -derivedDataPath .derivedData-macos-SFM \
  -skipPackagePluginValidation \
  -skipMacroValidation \
  CODE_SIGNING_ALLOWED=NO
```

On 2026-05-31, after rebuilding `Libbox.xcframework` from `v1.14.0-alpha.27` with the full Apple target, this completed with:

```text
** BUILD SUCCEEDED **
```

The local app product was generated at:

```text
<repo-root>/.derivedData-macos-SFM/Build/Products/Release/sing-box.app
```

The app executable is a universal macOS binary:

```text
x86_64
arm64
```

This command verifies compilation only. A distributable macOS archive still needs signing, entitlements, and notarization checks.

## Verified Signed macOS Build Command

From:

```text
<repo-root>
```

Run:

```bash
xcodebuild build \
  -project sing-box.xcodeproj \
  -scheme SFM \
  -configuration Release \
  -destination 'generic/platform=macOS' \
  -derivedDataPath .derivedData-macos-SFM-signed-check \
  -skipPackagePluginValidation \
  -skipMacroValidation
```

On 2026-05-31, this completed with:

```text
** BUILD SUCCEEDED **
```

Xcode used:

```text
Apple Development signing identity from the local keychain
Mac Team Provisioning Profile: io.vince.sfavt
```

The signed app product was generated at:

```text
<repo-root>/.derivedData-macos-SFM-signed-check/Build/Products/Release/sing-box.app
```

Known warning:

```text
The CFBundleVersion of an app extension ('1') must match that of its containing parent app ('395').
```

This warning did not block the local signed build, but should be fixed before a formal macOS distribution archive.

## Verified Archive Command

This archive result was verified before switching the fork from upstream `main` to upstream `dev`. Re-run it on the current `dev` branch when a signed TestFlight archive is needed.

From:

```text
<repo-root>
```

Run:

```bash
xcodebuild archive \
  -project sing-box.xcodeproj \
  -scheme SFI \
  -configuration Release \
  -destination 'generic/platform=iOS' \
  -archivePath build/SFI.xcarchive \
  -skipPackagePluginValidation \
  -skipMacroValidation \
  -allowProvisioningUpdates
```

On 2026-05-31 this completed with:

```text
** ARCHIVE SUCCEEDED **
```

The archive was generated at:

```text
<repo-root>/build/SFI.xcarchive
```

## Known Local State

- The working copy may have uncommitted `Localizable.xcstrings` changes created by Xcode string catalog updates.
- Do not discard those changes unless the user explicitly asks.
- `build/` and derived data are generated outputs. Keep the archive if the user needs it; derived data can be deleted.

## macOS SFM Helper Notes

The official downloaded SFM app can show:

```text
Failed to install helper service
未能完成该操作。Operation not permitted
```

For the official app, the helper is:

```text
Label: io.nekohasekai.sfavt.helper
Mach service: 287TTNZF8L.io.nekohasekai.sfavt.helper
App bundle: /Applications/SFM.app
Helper executable: /Applications/SFM.app/Contents/Helpers/RootHelper
Helper plist: /Applications/SFM.app/Contents/Library/LaunchDaemons/io.nekohasekai.sfavt.helper.plist
```

Use non-destructive checks first:

```bash
launchctl print system/io.nekohasekai.sfavt.helper
sfltool dumpbtm | rg -n -C 5 'SFM|sfavt|nekohasekai|RootHelper|287TTNZF8L|helper'
osascript -e 'tell application "System Events" to get the name of every login item'
```

Also check System Settings > General > Login Items & Extensions, especially both "Open at Login" and "Allow in Background".

Important: `sfltool resetbtm` resets the Background Task Management database globally. It can remove or change unrelated login/background item state, so only use it as a last resort after warning the user. After using it, restore affected login/background items manually in System Settings.
