# Release

How a build reaches a user. The local development loop is in [development.md](development.md);
the signing identity itself is in [signing.md](signing.md).

Tinycast Sonoma is a frozen macOS 14 edition. There is one channel, no beta, and **no in-app
updater** — updates are Homebrew-only, through `brew upgrade --cask tinycast-sonoma`.

## Packaging a DMG locally

```sh
./Scripts/build-dmg.sh            # -> build/Tinycast-Sonoma-<version>.dmg
./Scripts/build-dmg.sh 0.5.7      # -> build/Tinycast-Sonoma-0.5.7.dmg
```

It builds a Release `Tinycast Sonoma.app` signed with `Tinycast Self-Signed` and packs it with an
`/Applications` symlink. Local builds are never notarized.

## Signing & Gatekeeper

A distributed build is signed with a Developer ID, notarized and stapled, so a downloaded DMG opens
without a Gatekeeper warning. Full details in [signing.md](signing.md).

## How users get a release

Every release publishes a `Tinycast-Sonoma-<version>.dmg`, which people install by hand or through
the `tinycast-sonoma` cask. A `Tinycast-Sonoma-<version>.zip` may travel alongside it; the zip is
produced with `ditto -c -k --keepParent --sequesterRsrc`, the only zip that leaves the code signature
verifiable.

The cask declares `auto_updates false` (or omits the flag), so Homebrew compares the installed
version against the cask and upgrades it normally.

## Pull request review

There is no CI workflow and no automated reviewer, so the whole bar in
[testing.md](testing.md#definition-of-done) is run locally before a PR is opened.

## Releasing

There is no release workflow in this fork. Build with `Scripts/build-dmg.sh` on a Mac with the
signing identity, run `Scripts/verify-signature.sh` against the finished app, and publish the DMG
by hand. `Scripts/package-release.sh` notarizes and staples when the notary credentials are in the
environment.

`Scripts/check-migrations.sh` is worth running first: a migration that a release already shipped
stops the run until it is deleted ([migrations.md](migrations.md)).

The Dictation helper derives its own identifier with a `.dictation` suffix; signature verification
checks that its bundle and signing identifiers agree and remain distinct from the main app.
