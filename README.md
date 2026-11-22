# Eidolon desktop updates

Signed macOS update downloads for Eidolon. This repository holds the update manifest and, as GitHub Releases, the release files. The application source is in a separate private repository.

The installed app checks this channel, verifies every update against the public key built into it, and installs only when you ask it to in **Settings → Updates**. Automatic checks never install anything.

## Current channel

| | |
|---|---|
| Version | `0.1.0-rc.3` (release candidate, GitHub prerelease) |
| Platform | macOS 14 or later, Apple Silicon (`darwin-aarch64`) |
| macOS signing | Apple Development signature; not Developer ID signed or notarized |
| Manifest | [`latest.json`](latest.json), read from `https://raw.githubusercontent.com/Charlie284/Eidolon-updates/main/latest.json` |
| Release | [`v0.1.0-rc.3`](https://github.com/Charlie284/Eidolon-updates/releases/tag/v0.1.0-rc.3) |

These builds are for the current private preview. They do not claim general-public release readiness. Because they are not notarized, macOS Gatekeeper may warn about or block a freshly downloaded DMG.

## What is in this repository

| Path | Purpose |
|---|---|
| `latest.json` | The update manifest the app reads. It names the newest version and, per platform, the archive URL and its signature. |
| `README.md`, `docs/` | This documentation. |
| GitHub Releases (`vX.Y.Z[-rc.N]`) | One release per version. Each has `Eidolon.app.tar.gz` (the update archive), `Eidolon.app.tar.gz.sig` (its updater signature), `Eidolon_<version>_aarch64.dmg` (the installer for a new install) and `SHA256SUMS`. |

No app data, credentials, or signing private keys are stored here. Released archives are never replaced: a new version gets a new release, and the manifest is moved to point at it.

## How an update works

```mermaid
sequenceDiagram
    participant App as Installed Eidolon
    participant Raw as raw.githubusercontent.com
    participant Rel as GitHub Releases
    App->>Raw: GET main/latest.json (HTTPS)
    Raw-->>App: version, notes, platforms.darwin-aarch64
    Note over App: Shows the update only if version is newer than the installed one
    App->>Rel: User selects install: GET Eidolon.app.tar.gz
    Note over App: Verifies the signature with the built-in public key;<br/>a bad signature stops the install
    Note over App: Checks the bundle (identifier, version, architecture, code signature),<br/>swaps it in and keeps the previous bundle beside it
    Note over App: Waits for old processes to exit and reopens through Launch Services
```

The app uses the [Tauri updater](https://v2.tauri.app/plugin/updater/). Local data, settings and update recovery state are kept across the swap. The previous bundle stays in a hidden `.eidolon-update-*` folder next to the installed app as a manual rollback copy; it is not restored automatically.

## Manifest format

`latest.json` is a [Tauri static update manifest](https://v2.tauri.app/plugin/updater/#static-json-file):

```json
{
  "version": "0.1.0-rc.3",
  "notes": "Release notes shown in Settings → Updates.",
  "platforms": {
    "darwin-aarch64": {
      "url": "https://github.com/Charlie284/Eidolon-updates/releases/download/v0.1.0-rc.3/Eidolon.app.tar.gz",
      "signature": "<contents of Eidolon.app.tar.gz.sig>"
    }
  }
}
```

| Field | Meaning |
|---|---|
| `version` | SemVer of the newest release. The app offers it only if it is newer than the installed version. |
| `notes` | Plain-text release notes. |
| `platforms.<target>-<arch>.url` | HTTPS URL of the immutable release archive. Only `darwin-aarch64` is published. |
| `platforms.<target>-<arch>.signature` | The full text of the archive's `.sig` file (a base64-encoded minisign signature). |

## Verifying a download yourself

Check the published hashes from the folder you downloaded the release files into:

```sh
shasum -a 256 -c SHA256SUMS --ignore-missing
```

The update signature can be checked with [minisign](https://jedisct1.github.io/minisign/). The `.sig` file is a base64-encoded minisign signature, and the updater public key built into the app (key ID `FCF4561D2DFA02FD`) is:

```
RWT9AvotHVb0/ObeMYYL8Cc6T7bwnF65CYM/80ShDi824Xy4URAugnqU
```

```sh
base64 -d < Eidolon.app.tar.gz.sig > Eidolon.app.tar.gz.minisig
minisign -Vm Eidolon.app.tar.gz -x Eidolon.app.tar.gz.minisig \
  -P RWT9AvotHVb0/ObeMYYL8Cc6T7bwnF65CYM/80ShDi824Xy4URAugnqU
```

A valid signature's trusted comment names the file, for example `timestamp:1790616568	file:Eidolon.app.tar.gz` for `0.1.0-rc.3`.

## Publishing a release

See [docs/publishing.md](docs/publishing.md). In short: build and sign in the private source repository, create a new versioned release here with the four files, then update `latest.json` on `main` last.
