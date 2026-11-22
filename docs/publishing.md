# Publishing a release

Releases are built and signed from the private Eidolon source repository, which has the full build, manifest and verification tooling. This repository only receives the finished files. The steps below are the part that happens here.

## Rules

- **One release per version.** Never replace or re-upload an archive in an existing release. Installed apps and old manifests may still point at it.
- **Update `latest.json` last.** Apps start downloading as soon as `main/latest.json` changes, so every file it points to must already be published.
- **Keep the same updater key.** Installed apps only accept archives signed with the key whose public half is built into them. A different key makes every installed copy refuse the update.
- **Never commit secrets.** The signing private key, its password and any tokens stay outside this repository and its releases.
- **Use HTTPS release URLs only**, without embedded credentials.

## Steps

1. In the source repository, bump the version, build the app and produce `Eidolon.app.tar.gz`, its `.sig`, and the DMG. The source repository's release tooling checks that the archive's version, architecture and signature match before you continue.
2. Write `SHA256SUMS` for the archive, the `.sig` and the DMG:

   ```sh
   shasum -a 256 Eidolon.app.tar.gz Eidolon.app.tar.gz.sig Eidolon_<version>_aarch64.dmg > SHA256SUMS
   ```

3. Create the release here with tag `v<version>`. Mark release candidates as prereleases; the app reads `latest.json` directly, not GitHub's "latest release", so a prerelease is still delivered.

   ```sh
   gh release create v<version> --repo Charlie284/Eidolon-updates --prerelease \
     --title "Eidolon <version>" --notes-file notes.md \
     Eidolon.app.tar.gz Eidolon.app.tar.gz.sig Eidolon_<version>_aarch64.dmg SHA256SUMS
   ```

4. Check the uploaded files: download them from the release and run the checks in [Verifying a download yourself](../README.md#verifying-a-download-yourself).
5. Generate `latest.json` with the source repository's manifest tool, using the release's download URL (`https://github.com/Charlie284/Eidolon-updates/releases/download/v<version>/Eidolon.app.tar.gz`). The `signature` value must be the exact contents of the published `.sig`.
6. Commit `latest.json` to `main` and push. From this point installed apps see the update.
7. Confirm the live manifest: `curl -fsSL https://raw.githubusercontent.com/Charlie284/Eidolon-updates/main/latest.json`. The raw URL can be cached for a few minutes.

## Rolling back the channel

Installed apps only move to a **newer** version, so pointing `latest.json` back at an older release does not downgrade anyone; it only stops further installs of the bad version. To fix a bad release, publish a new, higher version and point the manifest at it. Leave the bad release's files in place.
