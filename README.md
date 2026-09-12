# Hylo Compiler Development Environment

A containerized development environment for the Hylo compiler.

By default, this image contains:

- Swift 6.3.3 (including Apple clang)
- LLVM 23.1.0 MinSizeRel with assertions enabled (see [Hylo build](https://github.com/hylo-lang/llvm-build) for which
  components are installed)
- [pkg-config](https://linux.die.net/man/1/pkg-config)
- LLVM's `bin/` folder on `PATH`
- LLVM's pkg-config file on `PKG_CONFIG_PATH` (package identifier: `llvm`)
- `LLVM_DIR` pointing at LLVM's CMake package directory

## Quick Start

```Dockerfile
# Specific version (recommended, for build reproducibility)
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:v3.1.0
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:v3.1.0-MinSizeRel
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:v3.1.0-Debug

# Latest stable release
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:latest
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:latest-MinSizeRel
FROM ghcr.io/hylo-lang/hylo-dev-toolchain:latest-Debug
```

`Debug`/`MinSizeRel` refers to the LLVM build contained in the image. The unsuffixed tags are `MinSizeRel`.


## Supported platforms
The image is multi-architecture, supporting `arm64` and `amd64`.

### Available tags

| Tag                              | Meaning                                                                                  |
|----------------------------------|------------------------------------------------------------------------------------------|
| `latest`, `latest-<BuildType>`   | Newest stable release.                                                                   |
| `vX.Y.Z`, `vX.Y.Z-<BuildType>`   | A specific release.                                                                      |
| `vX.Y.Z-beta.N`, `…-<BuildType>` | A prerelease (see [Prereleases](#prereleases)).                                          |
| `vX.Y.Z-<BuildType>-<arch>`      | Single-architecture build, kept for debugging. Prefer the multi-architecture tags above. |

### Optional build arguments

- `HYLO_LLVM_BUILD_TYPE`: `Debug` or `MinSizeRel` (default).
- `HYLO_LLVM_BUILD_RELEASE`: release tag from [llvm-build](https://github.com/hylo-lang/llvm-build/releases).
- `HYLO_LLVM_VERSION`: LLVM version, which must match the build named by the release tag, e.g. `23.1.0`.
