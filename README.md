# Tessl Snap

This repository contains the Snapcraft configuration to package [Tessl](https://tessl.io) for the Snap Store.

## Automated Version Sync

This repository uses GitHub Actions to automatically check for new upstream Tessl releases and update the snap package accordingly. The snap is a [classic-confined](https://snapcraft.io/docs/snap-confinement) snap that bundles the prebuilt Linux binary published at `install.tessl.io`.

### How it works

1. **Version Check**: Daily at 10:00 UTC, a GitHub Action fetches https://get.tessl.io and extracts the embedded `TESSL_VERSION`
2. **Auto Update**: If a new version is detected, it creates a pull request updating `snap/snapcraft.yaml`
3. **Build**: Merging the PR triggers `build-and-publish.yml`, which downloads the matching `tessl-<version>-linux-<arch>.tar.gz` from `install.tessl.io`, builds the snap, smoke-tests it, and publishes to the Snap Store edge channel

## Setup Instructions

### Prerequisites

1. Fork or clone this repo to `github.com/popey/tessl-snap`
2. Create a [GitHub Personal Access Token (PAT)](https://github.com/settings/tokens) with `repo` scope

### Set up GitHub Secrets

- `GH_TOKEN`: A GitHub Personal Access Token with `repo` scope, used by the sync workflow to open PRs
- `SNAPCRAFT_STORE_CREDENTIALS`: Snapcraft credentials, exported with:
  ```bash
  snapcraft export-login --snaps=tessl --channels=edge,stable snapcraft-token
  cat snapcraft-token
  ```
  Add the entire content as a secret.

### Workflow

- `build-and-publish.yml` runs on every push to `main`, builds the snap, smoke-tests it, and publishes to the `edge` channel.
- Promote to stable when ready: `snapcraft promote tessl --from-channel edge --to-channel stable`.

## Manual Build (Local Development)

If you want to build locally:

```bash
# Build using Launchpad (for multiple architectures)
snapcraft remote-build --launchpad-accept-public-upload

# Upload to the store
export SNAPCRAFT_STORE_CREDENTIALS=$(cat snapcraft_creds)
for f in tessl*.snap; do snapcraft upload "$f"; done

# Or build locally (your architecture only)
snapcraft

# Install for testing (classic confinement)
sudo snap install --classic --dangerous tessl_*.snap
```

## Workflows

### sync-version-with-upstream.yml
- **Schedule**: Runs daily at 10:00 UTC
- **Trigger**: Can also be manually triggered from Actions tab
- **Function**: Reads the embedded `TESSL_VERSION` from https://get.tessl.io and opens a PR if it differs from `snap/snapcraft.yaml`

### build-and-publish.yml
- **Trigger**: Runs on push to `main` branch
- **Function**: Builds snap in GitHub Actions, runs tests, and publishes to Snap Store edge channel

### test-snap-can-build.yml
- **Trigger**: Runs on pushes and pull requests against `main`
- **Function**: Builds the snap and runs `snapcraft-review-action` to catch packaging issues before merge

## Architecture

The snap supports:
- amd64 (x86_64)
- arm64 (aarch64)

## References

- Inspired by [Snapcrafters](https://github.com/snapcrafters) automation patterns
- [Discord snap](https://github.com/snapcrafters/discord) - automated version sync example
- [Signal Desktop snap](https://github.com/snapcrafters/signal-desktop) - full CI/CD example
- [Snapcrafters Guide](https://forum.snapcraft.io/t/automatic-snap-refreshes-using-github-actions-snapcrafters-guide/33444)
