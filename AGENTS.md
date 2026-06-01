# Agent Notes

This fork is maintained for VinceBox iOS/TestFlight work.

## Priority

- Focus on iOS first. Do not spend time on macOS, tvOS, or system-extension packaging unless the user explicitly asks.
- Follow `SagerNet/sing-box-for-apple` `dev`, not `main`.
- Keep local changes small and easy to rebase. Prefer signing, entitlement, bundle identifier, build, and documentation changes over source changes.
- Avoid changing app source files unless the user asks for a feature or bug fix. In particular, avoid unrelated edits under app/framework source areas such as `SFI`, `SFM`, `Extension`, `Library`, `ApplicationLibrary`, `Frameworks`, and Swift package code.
- Do not remove or overwrite uncommitted user changes. `Localizable.xcstrings` may contain local Xcode-generated changes.

## macOS Helper Caution

- The official downloaded SFM may fail to install its helper with `Operation not permitted` when macOS Background Items / ServiceManagement state is stale.
- Check `launchctl print system/io.nekohasekai.sfavt.helper`, `sfltool dumpbtm`, and System Settings > General > Login Items & Extensions before changing anything.
- Treat `sfltool resetbtm` as a last resort only. It resets macOS Background Task Management state globally and can affect unrelated "Open at Login" / "Allow in Background" entries.
- If `sfltool resetbtm` is used, re-check and manually restore the user's login/background items afterwards.

## Local Signing

- Apple Team ID is configured in the Xcode project build settings. Avoid recording personal account identifiers in docs.
- Main iOS bundle ID: `io.vince.sfavt.app`
- Packet tunnel extension: `io.vince.sfavt.app.extension`
- Intents extension: `io.vince.sfavt.app.intents`
- Widget extension: `io.vince.sfavt.app.widget`
- File provider extension: `io.vince.sfavt.app.fileprovider`
- App group: `group.io.vince.sfavt`
- iCloud container: `iCloud.io.vince.sfavt`

For iOS targets, keep `BASE_PACKAGE_IDENTIFIER = io.vince.sfavt.app` in sync with the main app bundle ID. The app derives runtime identifiers such as the packet tunnel provider, widget control kind, file provider domain, background task ID, and profile UTType from `BasePackageIdentifier`. Keep `APP_GROUP_IDENTIFIER = group.io.vince.sfavt` and `iCloud.io.vince.sfavt` unchanged unless the user explicitly asks to isolate containers.

The iOS packet tunnel extension currently does not include the Multicast Networking entitlement. If Apple approves the entitlement later, re-add `com.apple.developer.networking.multicast` only where it is actually needed and regenerate the provisioning profile.

## Build Notes

- `Libbox.xcframework` is required locally and is ignored by Git.
- For upstream `dev` at `1.14.0-alpha.27`, use a `Libbox.xcframework` built from `SagerNet/sing-box` tag `v1.14.0-alpha.27`.
- The local libbox was rebuilt with the full Apple target on 2026-05-31 and includes iOS, iOS simulator, tvOS, tvOS simulator, and macOS universal slices.
- An unsigned iOS Release build succeeded on 2026-05-31 with scheme `SFI` after replacing libbox.
- An unsigned macOS Release build succeeded on 2026-05-31 with scheme `SFM` after replacing libbox.
- A signed macOS Release build also succeeded on 2026-05-31 with scheme `SFM` and Apple Development signing.
- A signed iOS archive also succeeded earlier on 2026-05-31 with scheme `SFI` before switching the fork from upstream `main` to upstream `dev`.
- Use `docs/ios-testflight-maintenance.md` for the full sync, rebase, and archive workflow.
