# Rotko Networks F-Droid Repository

Self-hosted F-Droid repository for **[Zafu Zigner](https://github.com/rotkonetworks/zigner)**,
the air-gapped hardware signer for Zcash, Penumbra and Substrate.

Served from <https://foss.rotko.net> (haproxy → nginx container on
`web.rotko.net`), deployed by the `deploy` workflow in this repository.

## Add the repo to F-Droid

On Android, with the F-Droid app installed, tap:

> `fdroidrepos://foss.rotko.net/fdroid/repo?fingerprint=d80e4b0a4f13b483474717373c3b00859c15d370f9f39a513afc4b80299a800c`

Or manually — **F-Droid → Settings → Repositories → +**:

- **URL**: `https://foss.rotko.net/fdroid/repo`
- **Fingerprint**: `d80e4b0a4f13b483474717373c3b00859c15d370f9f39a513afc4b80299a800c`

## Why a self-hosted repo?

- Zigner's build compiles Rust to a WebAssembly module before gradle packages the
  APK. f-droid.org can be made to do this, but their build lag is measured in
  weeks; here a release is installable as soon as the tag's CI finishes.
- The APKs published here are byte-identical to the ones on the GitHub releases
  page and carry the same signing key, so switching between the two sources does
  not force a reinstall.

## Publishing a new release

Requires `fdroidserver` and the index keystore (see *Signing keys* below).

```sh
# 1. drop the release APK into fdroid/repo/
gh release download vX.Y.Z -R rotkonetworks/zigner \
  -p 'zigner-*.apk' -D fdroid/repo/

# 2. regenerate and sign the index
#    (no --clean: it wipes repo/icons/icon.png and the next index ships a
#     placeholder repo icon instead)
cd fdroid && fdroid update --pretty

# 3. commit and push — the deploy workflow rsyncs it to foss.rotko.net
git add -A && git commit -m "release: zigner vX.Y.Z" && git push
```

`fdroid update` moves anything beyond the six most recent versions into
`fdroid/archive/`, which is published as a second repo at
`https://foss.rotko.net/fdroid/archive`.

## Signing keys

Two different keys are involved — do not confuse them.

- **APK signing key**: the release signing key of the `zigner` app. Clients trust
  it; rotating it forces every user to uninstall and reinstall.
- **Index signing key**: signs this repository's index only. Rotating it forces
  clients to re-add the repo, but does **not** invalidate installed APKs.

Neither key is ever committed — `.gitignore` covers `config.yml`, `keystore.p12`
and `*.jks`. Both are held offline by the maintainers.
