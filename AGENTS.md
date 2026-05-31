# Agent Notes

This fork is maintained for VinceBox iOS/TestFlight work.

## Priority

- Focus on iOS first. Do not spend time on macOS, tvOS, or system-extension packaging unless the user explicitly asks.
- Follow `SagerNet/sing-box-for-apple` `dev`, not `main`.
- Keep local changes small and easy to rebase. Prefer signing, entitlement, bundle identifier, build, and documentation changes over source changes.
- Avoid changing app source files unless the user asks for a feature or bug fix. In particular, avoid unrelated edits under app/framework source areas such as `SFI`, `SFM`, `Extension`, `Library`, `ApplicationLibrary`, `Frameworks`, and Swift package code.
- Do not remove or overwrite uncommitted user changes. `Localizable.xcstrings` may contain local Xcode-generated changes.

## Local Signing

- Apple Team ID: `L922897LKX`
- Main iOS bundle ID: `io.vince.sfavt`
- Packet tunnel extension: `io.vince.sfavt.extension`
- Intents extension: `io.vince.sfavt.intents`
- Widget extension: `io.vince.sfavt.widget`
- File provider extension: `io.vince.sfavt.fileprovider`
- App group: `group.io.vince.sfavt`
- iCloud container: `iCloud.io.vince.sfavt`

The iOS packet tunnel extension currently does not include the Multicast Networking entitlement. If Apple approves the entitlement later, re-add `com.apple.developer.networking.multicast` only where it is actually needed and regenerate the provisioning profile.

## Build Notes

- `Libbox.xcframework` is required locally and is ignored by Git.
- A signed iOS archive succeeded on 2026-05-31 with scheme `SFI`.
- Use `docs/ios-testflight-maintenance.md` for the full sync, rebase, and archive workflow.
