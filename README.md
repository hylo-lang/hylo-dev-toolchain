# Hylo Compiler Development Environment

A containerized development environment for the Hylo compiler.

By default, this image contains:
- Swift 6.3.3
- LLVM 23.1.0 MinSizeRel with assertions enabled (see [Hylo build](https://github.com/hylo-lang/llvm-build) for which components are installed)
- [pkg-config](https://linux.die.net/man/1/pkg-config)
- LLVM's `bin/` folder on `PATH`
- LLVM's pkg-config file on `PKG_CONFIG_PATH` (package identifier: `llvm`)
- `LLVM_DIR` pointing at LLVM's CMake package directory

## Supported platforms

Images are multi-architecture: `linux/amd64` and `linux/arm64`. Docker picks the right one
automatically, so an Apple Silicon or ARM server pulls a native image with no emulation.

## Quick Start

```Dockerfile
# Specific version (recommended, for build reproducibility)
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:v3.0.0
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:v3.0.0-MinSizeRel
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:v3.0.0-Debug

# Latest stable release
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:latest
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:latest-MinSizeRel
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:latest-Debug
```

`Debug`/`MinSizeRel` refers to the LLVM build contained in the image. The unsuffixed tags are
`MinSizeRel`.

### Available tags

| Tag | Meaning |
| --- | --- |
| `latest`, `latest-<BuildType>` | Newest stable release. Never moved by a prerelease. |
| `vX.Y.Z`, `vX.Y.Z-<BuildType>` | A specific release. |
| `vX.Y.Z-beta.N`, `…-<BuildType>` | A prerelease — see [Prereleases](#prereleases). |
| `vX.Y.Z-<BuildType>-<arch>` | Single-architecture build, kept for debugging. Prefer the multi-architecture tags above. |

### Optional build arguments

- `HYLO_LLVM_BUILD_TYPE`: `Debug` or `MinSizeRel` (default).
- `HYLO_LLVM_BUILD_RELEASE`: release tag from [llvm-build](https://github.com/hylo-lang/llvm-build/releases).
- `HYLO_LLVM_VERSION`: LLVM version, which must match the build named by the release tag, e.g. `23.1.0`.

## CI

A single workflow, [`.github/workflows/ci.yml`](.github/workflows/ci.yml), covers both pull
requests and releases.

Every run builds all four image variants — `{MinSizeRel, Debug} × {amd64, arm64}` — each on a
runner of its own architecture, so nothing is emulated. Each image is then verified by building
and testing [Swifty-LLVM](https://github.com/hylo-lang/Swifty-LLVM) *inside it* with SwiftPM,
plus a check that `LLVM_DIR` points at a usable `LLVMConfig.cmake`. That exercises what actually
matters: `pkg-config` resolving `llvm` and linking against LLVM.

Swifty-LLVM is pinned to a commit via `SWIFTY_LLVM_REF` in the workflow, so an unrelated upstream
breakage cannot turn this repository red.

**Pull requests build and verify, but never push.** Nothing reaches the registry until a release
tag is pushed, which is also why pull requests from forks work here.

## Releasing

Push a tag:

```bash
git tag v3.1.0
git push origin v3.1.0
```

That single action drives the whole release:

1. A **draft release** is created, with notes generated from the merged pull requests since the
   previous tag.
2. All four images are built and verified, exactly as on a pull request.
3. Each image is pushed as `vX.Y.Z-<BuildType>-<arch>` — only after *it* has passed.
4. Once **all four** have passed, the multi-architecture tags (`vX.Y.Z`, `vX.Y.Z-<BuildType>`,
   `latest`, `latest-<BuildType>`) are assembled, and the release is published.

The multi-architecture tags are assembled from the image *digests* this run pushed, never from
the per-architecture tags, so an image left behind by an earlier attempt cannot end up in a
published manifest; re-pushing a moved tag rebuilds and overwrites every image.

An existing draft release is left untouched, so if you move a tag to a different commit, delete
its draft release first to get notes describing the new one.

If any variant fails, the release stays a draft and no consumable tag is created. Use
**Re-run failed jobs** — only the failed variant is rebuilt. There is no need to re-push the tag.

Tags must be `vX.Y.Z` or `vX.Y.Z-<prerelease>`; anything else fails the run immediately.

### Prereleases

Any tag with a suffix — `v4.0.0-beta.1` — is a prerelease. It behaves the same, with two
differences:

- **Verification does not block the push.** The images are published even if Swifty-LLVM fails to
  build against them, and the failure is recorded in the release notes.
- **`latest` and `latest-<BuildType>` are not moved.**

This exists to break the circular dependency on a breaking change. When bumping to a new LLVM
major version, Swifty-LLVM cannot compile against the new toolchain until it has been updated, and
it cannot be updated until an image exists. So:

1. Push `v4.0.0-beta.1` to publish a candidate image.
2. Point Swifty-LLVM's devcontainer at it and fix it there.
3. Update `SWIFTY_LLVM_REF` in this repository to the fixed commit.
4. Push `v4.0.0`, which now passes the full gate.

For a change that needs a coordinated Swifty-LLVM update but no published image, point
`SWIFTY_LLVM_REF` at the Swifty-LLVM branch in the same pull request instead, and move it to a
commit on `main` before releasing.

## Contributing

1. Create a branch from `main`.
2. Make your changes.
3. Open a pull request.
4. Merge when approved.

Labels no longer affect anything — you choose the version by choosing the tag, and release notes
are GitHub's default generated list of merged pull requests. Edit the draft by hand before
publishing if a release needs more than that.
