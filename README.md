# Eidolon desktop updates

Signed macOS update downloads for Eidolon. This repository contains release files and the update manifest; the application source repository remains private.

The installed application verifies every update against its embedded public key before installation. Updates are installed only when requested in Settings → Updates.

## Current channel

This is the macOS Apple Silicon release-candidate channel. These builds use an Apple Development signature, not Developer ID notarization. They are intended for the current private preview and are not a general-public release-readiness claim.

The updater manifest is `latest.json`. Versioned release archives are immutable; a newer manifest points to the next signed archive. No app data, credentials, or signing private keys are stored here.
