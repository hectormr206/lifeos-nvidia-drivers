# lifeos-nvidia-drivers

NVIDIA proprietary driver RPMs for [LifeOS](https://github.com/hectormr206/lifeos), packaged as an OCI image and consumed by the main LifeOS Containerfile via `COPY --from=...`.

Fork of [bazzite-org/nvidia-drivers](https://github.com/bazzite-org/nvidia-drivers) — itself a mirror of [negativo17's nvidia-driver](https://github.com/negativo17/nvidia-driver) plus a GitHub Actions pipeline that builds and publishes OCI images.

## What this ships

- Current version: whatever `nvidia-driver/nvidia-driver.spec` has in `Version:` (bumped via submodule update). Today: **595.58.03** (NVIDIA Production Branch).
- Target: Fedora 43, `x86_64`. (No `aarch64`, no `580 legacy` — LifeOS doesn't need them today.)
- Packages inside the OCI image (flat layout, all `.rpm` under `/`):
  - `nvidia-kmod-common`, `nvidia-kmod` (pre-built kernel modules for Fedora's shipping kernels)
  - `nvidia-driver`, `nvidia-driver-selinux`
  - `nvidia-modprobe`, `nvidia-settings`, `nvidia-persistenced`
  - `nvidia-container-toolkit` + `libnvidia-container`

## How LifeOS uses it

LifeOS's `image/Containerfile` mounts this image and `dnf install`s the RPMs in the final stage. The existing kmod signing logic in LifeOS then signs the shipped `.ko` files with the `LifeOS NVIDIA Kmod Secure Boot` MOK before sealing the image.

## Tags published to GHCR

- `ghcr.io/hectormr206/lifeos-nvidia-drivers:<driver-version>-f43-x86_64` — immutable, version-pinned.
- `ghcr.io/hectormr206/lifeos-nvidia-drivers:latest-f43-x86_64` — moves on every `master` build and on the weekly cron.

## When does it rebuild?

- Every push to `master` (e.g. bumping the `nvidia-driver` submodule).
- Weekly cron on Mondays 05:17 UTC — catches new Fedora kernel-devel packages so kmods stay compatible.
- Manual `workflow_dispatch` with optional `nvidia-version` override.

## Bumping the NVIDIA version

1. Find a commit of `bazzite-org/nvidia-driver` (or `negativo17/nvidia-driver` if you want to track upstream directly) whose `nvidia-driver.spec` has the `Version:` you want.
2. `cd nvidia-driver && git fetch && git checkout <sha>` and commit the submodule update from the root.
3. Push to `master`. CI builds and publishes.

Or, for a one-off build without touching the submodule pin, trigger `workflow_dispatch` with `nvidia-version: 5XX.YY.ZZ`.

## License

NVIDIA proprietary driver — distributed under NVIDIA's own EULA terms (same as negativo17 and Bazzite have done for years). Spec files and build glue are GPLv2 per negativo17's originals.
