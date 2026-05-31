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
L922897LKX
io.vince.sfavt
group.io.vince.sfavt
iCloud.io.vince.sfavt
```

This matters because upstream `dev` may add new targets, build phases, or package references.

## Vince Identifiers

Apple Team ID:

```text
L922897LKX
```

Bundle identifiers:

```text
io.vince.sfavt
io.vince.sfavt.extension
io.vince.sfavt.intents
io.vince.sfavt.standalone
io.vince.sfavt.system
io.vince.sfavt.widget
io.vince.sfavt.fileprovider
```

Shared containers:

```text
group.io.vince.sfavt
iCloud.io.vince.sfavt
```

## Multicast Networking

The original iOS packet tunnel extension requested:

```text
com.apple.developer.networking.multicast
```

This entitlement requires Apple approval. It is not required for the basic Packet Tunnel VPN entitlement, so it was removed from `Extension/Extension.entitlements` to let the iOS archive sign without waiting for Apple approval.

If Apple approves Multicast Networking later:

1. Enable the capability for the relevant App ID, most likely `io.vince.sfavt.extension`.
2. Regenerate or refresh the provisioning profile.
3. Re-add `com.apple.developer.networking.multicast` only to the target that needs it.
4. Re-run a signed archive.

## Libbox

`Libbox.xcframework` is required but ignored by Git. If it is missing, build it from the matching `SagerNet/sing-box` version.

For the previously verified iOS archive, `Libbox.xcframework` was present at:

```text
/Users/yunshi/codes/sing-box-for-apple/Libbox.xcframework
```

It contained:

```text
ios-arm64
ios-arm64_x86_64-simulator
```

## Verified Archive Command

From:

```text
/Users/yunshi/codes/sing-box-for-apple
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
/Users/yunshi/codes/sing-box-for-apple/build/SFI.xcarchive
```

## Known Local State

- The working copy may have uncommitted `Localizable.xcstrings` changes created by Xcode string catalog updates.
- Do not discard those changes unless the user explicitly asks.
- `build/` and derived data are generated outputs. Keep the archive if the user needs it; derived data can be deleted.
