# homelab-ucore &nbsp; [![bluebuild build badge](https://github.com/exonomos/homelab-ucore/actions/workflows/build.yml/badge.svg)](https://github.com/exonomos/homelab-ucore/actions/workflows/build.yml)

A custom [Fedora uCore](https://github.com/ublue-os/ucore) image for a home server, built with [BlueBuild](https://blue-build.org/). Automatically built every night via GitHub Actions and signed with [cosign](https://github.com/sigstore/cosign).

## Features

| Area | Detail |
|------|--------|
| **Base** | `ghcr.io/ublue-os/ucore:stable` — immutable, atomic Fedora CoreOS |
| **DNS** | Quad9 (`9.9.9.9`, `149.112.112.112`) via systemd-resolved |
| **ZRAM** | Compressed swap in RAM (`ram / 2`) |
| **Swap** | 8 GiB swap file at `/var/swapfile` |
| **Keyboard** | German layout (`de`) |
| **Early SSH** | `dracut-sshd` with authorized key for remote LUKS unlock |
| **Container network** | `slirp4netns` as default rootless backend |
| **Updates** | Staged automatically (`AutomaticUpdatePolicy=stage`) |
| **Cockpit** | Web-based admin UI enabled |
| **Zincati** | Disabled and masked — manual update control |
| **Packages** | `neovim`, `dracut-sshd` |
| **CI** | Nightly build at 06:00 UTC via GitHub Actions, cosign-signed |

## How It Works

This repository uses [BlueBuild](https://blue-build.org/) to define a custom container-native OS image. The [`recipe.yml`](./recipes/recipe.yml) declares the base image and modules — files dropped into `/etc`, extra RPM packages, and systemd service enablement. GitHub Actions runs the [`blue-build/github-action`](https://github.com/blue-build/github-action) every night, which builds a new OCI image and pushes it to the GitHub Container Registry at `ghcr.io/exonomos/ucore`.

The result is a ready-to-use Fedora CoreOS derivative you can rebase any existing Atomic Fedora installation onto.

## Installation

> [!WARNING]
> [This is an experimental feature](https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable), try at your own discretion.

To rebase an existing Atomic Fedora installation to the latest build:

1. Rebase to the unsigned image to get the proper signing keys and policies installed:
   ```bash
   rpm-ostree rebase ostree-unverified-registry:ghcr.io/exonomos/ucore:latest
   ```
2. Reboot:
   ```bash
   systemctl reboot
   ```
3. Rebase to the signed image:
   ```bash
   rpm-ostree rebase ostree-image-signed:docker://ghcr.io/exonomos/ucore:latest
   ```
4. Reboot again:
   ```bash
   systemctl reboot
   ```

The `latest` tag always points to the most recent nightly build and stays on the Fedora version specified in [`recipe.yml`](./recipes/recipe.yml).

## Local Build

You can build the image locally for testing:

```bash
pip install bluebuild
bluebuild build recipes/recipe.yml
```

See the [BlueBuild docs](https://blue-build.org/) for details.

## Verification

Images are signed with [Sigstore](https://www.sigstore.dev/)'s [cosign](https://github.com/sigstore/cosign). Verify the signature with the public key in this repo:

```bash
cosign verify --key cosign.pub ghcr.io/exonomos/ucore
```

## License

[Apache License 2.0](./LICENSE)
