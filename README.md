# GPG Import Action

[![GitHub Action](https://img.shields.io/badge/GitHub_Action-purple?logo=github&logoColor=white)](https://github.com/purpleclay/gpg-import-action)
[![MIT](https://img.shields.io/badge/MIT-gray?logo=github&logoColor=white)](LICENSE)

Import a GPG key and configure git commit and tag signing, non-interactively, using [gpg-import](https://github.com/purpleclay/gpg-import). Runs on Linux and macOS runners.

## Features

- Import GPG keys in ASCII armored format (_optionally base64 encoded for CI environments_).
- Pass the key and passphrase to `gpg-import` as environment variables, keeping them off the command line.
- Preset the passphrase in the GPG agent for the primary key and every subkey, for non-interactive signing.
- Configure local or global git signing for commits, tags and pushes, or only the ones you choose with the `sign` input. Settings for the rest are left alone.
- Select a specific key or subkey for signing by fingerprint. GPG then signs with exactly that key, not the newest signing subkey.
- Set key trust level.
- Verify the imported key can sign before configuring git, so a wrong passphrase fails the step rather than the first commit.
- Refuse an expired primary key or selected subkey, and flag unused expired subkeys in the output.
- Create `gpg.conf` and `gpg-agent.conf` with defaults for non-interactive signing, without overwriting existing ones.
- Override committer identity independently from the GPG key.
- Dry-run mode to preview changes without applying them.

## Preparing your GPG key

[Generate](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key) a GPG key and export it as an ASCII armored private key:

```sh
gpg --armor --export-secret-key batman@dc.com

# For CI systems that don't handle multiline secrets, base64 encode it.
# Wrapped output is accepted, so there's no need for -w 0
gpg --armor --export-secret-key batman@dc.com | base64
```

Export one key per run. If the input contains more than one key, public or secret, `gpg-import` lists their fingerprints and fails. By then, gpg has already added them all to the keyring.

Store the output as a repository secret (e.g. `GPG_PRIVATE_KEY`). If your key has a passphrase, store that as a separate secret (e.g. `GPG_PASSPHRASE`). The `key` input accepts both raw armored PGP and base64-encoded formats.

### Choosing the git identity

By default, git's `user.name` and `user.email` come from the key's first uid. To commit as another uid on the key, pass its email with the `git-committer-email` input, and its name is used as well. `git-committer-name` sets the name, whichever email you pass.

> [!NOTE]
> GitHub only shows a signed commit as Verified when the committer email matches one of the key's uids. `gpg-import` still accepts an email that isn't on the key, such as a bot or noreply address, but prints a warning listing the key's uids.

## Usage

### Import a GPG key and configure Git signing

Git signing is configured for the local repository by default, so check it out first.

```yaml
- uses: actions/checkout@v7

- uses: purpleclay/gpg-import-action@v0
  with:
    key: ${{ secrets.GPG_PRIVATE_KEY }}
    passphrase: ${{ secrets.GPG_PASSPHRASE }}
```

### Commit as another uid on the key

If your key has more than one uid, pass the email of the one to commit as. Its name is used too, so there's no need to set `git-committer-name`.

```yaml
- uses: purpleclay/gpg-import-action@v0
  with:
    key: ${{ secrets.GPG_PRIVATE_KEY }}
    passphrase: ${{ secrets.GPG_PASSPHRASE }}
    git-committer-email: batman@wayne-enterprises.com
```

### Sign with a specific subkey

GPG normally signs with the newest signing subkey. To use a different one, pass its full fingerprint, as listed by `gpg --list-secret-keys --with-subkey-fingerprint`.

```yaml
- uses: purpleclay/gpg-import-action@v0
  with:
    key: ${{ secrets.GPG_PRIVATE_KEY }}
    passphrase: ${{ secrets.GPG_PASSPHRASE }}
    fingerprint: 5D3A9C04B6E2F17A8C4D0E9B21F6A7C3D8E05B14
```

### Sign tags only

For a release job that signs its tags but not its version-bump commits. Commit and push signing settings are left as they are.

```yaml
- uses: purpleclay/gpg-import-action@v0
  with:
    key: ${{ secrets.GPG_PRIVATE_KEY }}
    passphrase: ${{ secrets.GPG_PASSPHRASE }}
    sign: tags
```

### Skip git configuration

For jobs that sign something other than git, such as release artifacts or packages. The key is imported and its passphrase preset, but git is left untouched.

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

    # Override the git committer email. If it matches another uid on the GPG
    # key, that uid's name is used too.
    # Optional.
    git-committer-email:

    # Preview changes without applying them.
    # Optional. Default is false
    dry-run:

    # Skip the test signature that proves the imported key can sign. Without
    # it, a wrong or missing passphrase fails the import.
    # Optional. Default is false
    skip-verify:

    # The git operations to sign, comma separated: any of commits, tags and
    # pushes. Settings for the others are left as they are.
    # Optional. Default is commits,tags,pushes
    sign:
```

## Outputs

| Output               | Description                              |
| -------------------- | ---------------------------------------- |
| `gpg-import-version` | The gpg-import version that was executed |
