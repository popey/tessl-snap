# Tessl Snap

This repository contains the Snapcraft configuration to package [Tessl](https://tessl.io) for the Snap Store.

## Automated Version Sync

This repository uses GitHub Actions to automatically check for new upstream releases of `@tessl/cli` on npm and update the snap package accordingly.

### How it works

1. **Version Check**: Every 6 hours, a GitHub Action checks the latest version of `@tessl/cli` on npm
2. **Auto Update**: If a new version is detected, it creates a pull request updating `snap/snapcraft.yaml`
3. **Build Options**:
   - **Option A (Simple)**: Hook this repo to [Snapcraft Build Service](https://snapcraft.io/build) - merging the PR triggers an automatic build
   - **Option B (Full CI)**: Use GitHub Actions to build, test, and publish to the Snap Store

## Setup Instructions

### Prerequisites

1. Fork or clone this repo to `github.com/popey/tessl-snap`
2. Create a [GitHub Personal Access Token (PAT)](https://github.com/settings/tokens) with `repo` scope

### Option A: Using Snapcraft Build Service (Recommended for simplicity)

This is the simpler approach - just let Snapcraft's infrastructure handle the building.

1. **Set up GitHub Secret**:
   - Go to your repo Settings → Secrets and variables → Actions
   - Add a new secret named `GH_TOKEN` with your GitHub PAT

2. **Connect to Snapcraft Build Service**:
   - Go to [snapcraft.io/build](https://snapcraft.io/build)
   - Connect your GitHub repo
   - Enable automatic builds on the `main` branch

3. **Done!** When the version sync workflow creates a PR and you merge it, Snapcraft will automatically build and publish

### Option B: Full GitHub Actions CI/CD

Build and test in GitHub Actions before publishing to the store.

1. **Set up GitHub Secrets**:
   - `GH_TOKEN`: Your GitHub Personal Access Token
   - `SNAPCRAFT_STORE_CREDENTIALS`: Export your Snapcraft credentials using:
     ```bash
     snapcraft export-login --snaps=tessl --channels=edge,stable snapcraft-token
     cat snapcraft-token
     ```
     Add the entire content as a secret

2. **Enable the workflow**:
   - The `build-and-publish.yml` workflow will run on every push to `main`
   - It builds the snap, tests it, and publishes to the `edge` channel

3. **Manual promotion**:
   - Test the edge version
   - Manually promote to stable when ready: `snapcraft promote tessl --from-channel edge --to-channel stable`

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

# Install for testing
sudo snap install --dangerous tessl_*.snap
```

## Workflows

### sync-version-with-upstream.yml
- **Schedule**: Runs every 6 hours
- **Trigger**: Can also be manually triggered from Actions tab
- **Function**: Checks npm for new `@tessl/cli` versions and creates a PR if an update is available

### build-and-publish.yml (Optional)
- **Trigger**: Runs on push to `main` branch
- **Function**: Builds snap in GitHub Actions, runs tests, and publishes to Snap Store edge channel

## Architecture

The snap supports:
- amd64 (x86_64)
- arm64 (aarch64)

## References

- Inspired by [Snapcrafters](https://github.com/snapcrafters) automation patterns
- [Discord snap](https://github.com/snapcrafters/discord) - automated version sync example
- [Signal Desktop snap](https://github.com/snapcrafters/signal-desktop) - full CI/CD example
- [Snapcrafters Guide](https://forum.snapcraft.io/t/automatic-snap-refreshes-using-github-actions-snapcrafters-guide/33444)
