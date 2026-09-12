## CI

Pull requests build and verify the images on all platforms using a pinned version
of [Swifty-LLVM](https://github.com/hylo-lang/Swifty-LLVM).

## Releasing

Push a tag:

```bash
git tag v3.1.0
git push origin v3.1.0
```

Tags must be `vX.Y.Z` or `vX.Y.Z-...`; anything else fails the run immediately.

A draft release is created automatically. Upon successful verification of all parts of all images, they are pushed to
GHCR and the release is published.

### Prereleases

Any tag with a `-...` suffix, (e.g. `v4.0.0-beta`) is a prerelease. They are special because verification doesn't
prevent the image from being pushed to GHCR. This is here to break the circular dependency between SwiftyLLVM and this
repo.
