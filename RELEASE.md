# Releasing

This is a composite GitHub Action — there's nothing to build. Consumers use the source directly at a tagged git ref, so releasing is just: tag, verify, publish.

- [Versioning](#versioning)
- [Cutting a release](#cutting-a-release)
- [Moving the major version tag](#moving-the-major-version-tag)
- [Fixing a bad release](#fixing-a-bad-release)

## Versioning

Tags are `vX.Y.Z`. Consumers pin either a full commit SHA or the floating major tag (e.g. `v0`) — the convention used throughout this action's own README (`purpleclay/gpg-import-action@v0`). Semver tracks **the caller interface**:

- **Major** — renamed/removed inputs or outputs; any *increase* in the permissions callers must grant.
- **Minor** — new inputs with defaults, new outputs.
- **Patch** — dependency bumps, internal hardening, documentation.

## Cutting a release

Tagging is a manual step today — this repo doesn't yet use [`nsv`](https://github.com/purpleclay/nsv) to automate it, though that may change.

1. Confirm `main` is green (ci) and every merged-but-unreleased change is accounted for:

   ```sh
   last_tag=$(git describe --tags --abbrev=0 --match 'v*.*.*' 2>/dev/null || true)
   git log "${last_tag:+$last_tag..}main" --oneline
   ```

2. Create a signed, annotated tag locally and push it:

   ```sh
   git tag -s vX.Y.Z -m "chore: release for vX.Y.Z"
   git push origin vX.Y.Z
   ```

   The tag must be signed by an authorized signer (`purpleclay`) — the `release` workflow verifies this before publishing anything.

3. The `release` workflow triggers on the tag push. It validates the tag format, verifies the signature, generates release notes ([release-note-action](https://github.com/purpleclay/release-note-action)), and publishes the GitHub release. No build assets.

4. [Move the major version tag](#moving-the-major-version-tag).

5. Verify: the release exists, the notes render correctly, and `purpleclay/gpg-import-action@vX` resolves to the new commit.

## Moving the major version tag

The floating major tag (`vX`) isn't moved automatically — do it straight after the release above publishes:

```sh
git tag -sf vX -m "chore: release for vX.Y.Z"
git push origin vX --force
```

## Fixing a bad release

Never by mutation. A broken release is followed by a fixed patch release; the bad tag stays. Once the patch is out, move the major tag on to it as above.
