# GPG Import

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

| Input                 | Description                                             | Required | Default  |
| --------------------- | ------------------------------------------------------- | :------: | :------: |
| `version`             | The version of gpg-import to install                    |    No    | `latest` |
| `key`                 | The GPG private key to import                           |   Yes    |          |
| `passphrase`          | The passphrase for the GPG private key                  |    No    |          |
| `fingerprint`         | Select a specific key or subkey by fingerprint          |    No    |          |
| `trust-level`         | Set the trust level for the imported key (`1`-`5`)      |    No    |          |
| `skip-git`            | Skip git configuration after importing the key          |    No    | `false`  |
| `git-global-config`   | Apply git signing settings globally rather than locally |    No    | `false`  |
| `git-committer-name`  | Override the git committer name                         |    No    |          |
| `git-committer-email` | Override the git committer email                        |    No    |          |
| `dry-run`             | Preview changes without applying them                   |    No    | `false`  |
