# Zodl F-Droid Repository

Official self-hosted F-Droid repository for the **Zodl Zcash Wallet**, served
from GitHub Pages at <https://foss.zodl.com>.

Builds and publishing are automated — the `publish_to_fdroid` job in
[`zodl-inc/zodl-android`](https://github.com/zodl-inc/zodl-android) commits to
this repo on every GitHub Release.

## Add the repo to F-Droid client

On Android, tap this link with the official F-Droid app installed:

> `fdroidrepos://foss.zodl.com/fdroid/repo?fingerprint=7e751ab710159dff44f55631f910ba4033f0ebd2f867691af633cece4ddb62e4`

Or manually: **F-Droid → Settings → Repositories → +** and enter:

- **URL**: `https://foss.zodl.com/fdroid/repo`
- **Fingerprint**: `7e751ab710159dff44f55631f910ba4033f0ebd2f867691af633cece4ddb62e4`

> Replace `7e751ab710159dff44f55631f910ba4033f0ebd2f867691af633cece4ddb62e4` with the SHA-256 fingerprint of the index
> signing certificate (fixed once the keystore is generated — see
> `infra/terraform/environments/zodl-android-fdroid/`).

## Why not f-droid.org?

- f-droid.org builds APKs from source and cannot access build-time secrets
  (Flexa / CMC / Crashlytics keys). Our own repo builds with the full set.
- Faster release cadence — new versions appear here as soon as the release
  workflow completes, no fdroiddata PR lag.

## How it works

```
zodl-inc/zodl-android release.yaml
  └─ publish_to_fdroid
     ├─ pulls FOSS APK from the GitHub Release
     ├─ runs `fdroid update` (index signed with Secrets Manager keystore)
     └─ commits to this repo → GitHub Pages deploys to foss.zodl.com
```

## Signing keys

- **APK signing key**: same upload keystore as Google Play
  (`UPLOAD_KEYSTORE_BASE_64`). Never rotated casually — clients trust it.
- **Index signing key** (different!): `fdroid-index.jks` stored as JSON blob
  in AWS Secrets Manager at `/infra/zodl-android/fdroid_index_keystore`.
  Rotating this forces clients to re-add the repo, but does NOT invalidate
  installed APKs.

See `CLAUDE.md` (section *Zodl Android — F-Droid repo*) in the
[`zodl-inc/infra`](https://github.com/zodl-inc/infra) repository for the full
runbook.
