# GPG Import Action

[![GitHub Action](https://img.shields.io/badge/GitHub_Action-purple?logo=github&logoColor=white)](https://github.com/purpleclay/gpg-import-action)
[![MIT](https://img.shields.io/badge/MIT-gray?logo=github&logoColor=white)](LICENSE)

A GitHub Action that imports and configures a GPG key for Git signing using [gpg-import](https://github.com/purpleclay/gpg-import).

## Features

- Import GPG keys in ASCII armored format (_optionally base64 encoded for CI environments_).
- Configure local or global git signing settings.
- Preset GPG agent passphrase for non-interactive signing.
- Set key trust level.
- Select a specific key or subkey for signing via fingerprint.
- Override committer identity independently from the GPG key.
- Dry-run mode to preview changes without applying them.

## Exporting your GPG Key

If you don't have a GPG key, follow GitHub's guide to [generate](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key) one.

Export your GPG private key in ASCII armored format:

```sh
gpg --armor --export-secret-key batman@dc.com
```

For CI environments where multiline secrets are not supported, encode the key as base64:

```sh
gpg --armor --export-secret-key batman@dc.com | base64
```

Store the output as a repository secret (e.g. `GPG_PRIVATE_KEY`). If your key has a passphrase, store that as a separate secret (e.g. `GPG_PASSPHRASE`). The `key` input accepts both raw armored PGP and base64-encoded formats.

## Usage

### Import a GPG key and configure Git signing

```yaml
- uses: purpleclay/gpg-import-action@v0
  with:
    key: ${{ secrets.GPG_PRIVATE_KEY }}
    passphrase: ${{ secrets.GPG_PASSPHRASE }}
```

### Select a specific subkey with trust

```yaml
- uses: purpleclay/gpg-import-action@v0
  with:
    key: ${{ secrets.GPG_PRIVATE_KEY }}
    passphrase: ${{ secrets.GPG_PASSPHRASE }}
    fingerprint: ${{ secrets.GPG_FINGERPRINT }}
    trust-level: 5
```

### Skip git configuration

```yaml
- uses: purpleclay/gpg-import-action@v0
  with:
    key: ${{ secrets.GPG_PRIVATE_KEY }}
    passphrase: ${{ secrets.GPG_PASSPHRASE }}
    skip-git: true
```

## Inputs

See [action.yml](action.yml)

```yaml
- uses: purpleclay/gpg-import-action@v0
  with:
    # Version of gpg-import to download.
    # Optional. Defaults to the version this action release was tested
    # against, so pinning the action pins a known-good pairing.
    version:

    # GitHub token used to download the gpg-import release.
    # Optional. Default is github.token
    token:

    # Verify the downloaded gpg-import checksum and attestation before
    # execution. Requires gpg-import >= 0.10.0.
    # Optional. Default is true
    verify-attestation:

    # The GPG private key to import.
    # Required.
    key:

    # The passphrase for the GPG private key.
    # Optional.
    passphrase:

    # Select a specific key or subkey by fingerprint.
    # Optional.
    fingerprint:

    # Set the trust level for the imported key (1-5).
    # Optional.
    trust-level:

    # Skip git configuration after importing the key.
    # Optional. Default is false
    skip-git:

    # Apply git signing settings globally rather than locally.
    # Optional. Default is false
    git-global-config:

    # Override the git committer name instead of using the value from the
    # GPG key.
    # Optional.
    git-committer-name:

    # Override the git committer email instead of using the value from the
    # GPG key.
    # Optional.
    git-committer-email:

    # Preview changes without applying them.
    # Optional. Default is false
    dry-run:
```

## Outputs

| Output                | Description                               |
| ---------------------- | ------------------------------------------ |
| `gpg-import-version`  | The gpg-import version that was executed  |
