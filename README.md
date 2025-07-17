# Hylo Compiler Development Environment

A containerized development environment for the Hylo compiler with automated CI/CD pipeline.

## Quick Start

### Using Docker Images

```Dockerfile
# Specific version (recommended, for build reproducability)
FROM ghcr.io/tothambrus11/hylo-compiler-dev-env:v1.0.0

# Latest stable release
FROM ghcr.io/tothambrus11/hylo-compiler-dev-env:latest
```

## CI/CD Pipeline

### Overview

The repository uses an automated CI/CD system that:
- Builds Docker images for every pull request, with a temporary published package (`pr-...`)
- Creates draft releases automatically after merging
- Cleans up experimental images automatically

### Pull Request Workflow

1. **Open a PR** → Automatically builds a Docker image tagged as `pr-<number>-<sha>`.
2. **Test the PR image** using the instructions in the automatic PR comment added after a successful CI build 
3. **Merge or close PR** → Image is automatically cleaned up

### Release Workflow

1. **Push to main** → Updates draft release and builds draft image
2. **Publish release** → Builds final release image and updates `latest` tag
3. **Old images cleanup** → Runs weekly to remove experimental images >7 days old

## Contributing

### Commit Messages and PR Labels

Your commit messages should be clear and descriptive. However, the **PR labels** are what control versioning and release notes that can be added by maintainers.

#### Version Bumping Labels
- `major` or `breaking` → Major version bump (1.0.0 → 2.0.0)
- `minor`, `feature`, or `enhancement` → Minor version bump (1.0.0 → 1.1.0)  
- `patch`, `fix`, `bugfix`, or `bug` → Patch version bump (1.0.0 → 1.0.1)
- `chore` or `maintenance` → Patch version bump

#### Categorization Labels
- `feature` / `enhancement` → 🚀 Features
- `fix` / `bugfix` / `bug` → 🐛 Bug Fixes
- `chore` / `maintenance` → 🧰 Maintenance
- `documentation` / `docs` → 📚 Documentation
- `ci` / `cd` / `build` → 🏗️ CI/CD
- `dependencies` / `deps` → ⬆️ Dependencies

#### Special Labels
- `skip-changelog` → Excludes from release notes
- `duplicate` / `invalid` / `wontfix` → Excludes from release notes

### Development Process

1. **Create a feature branch** from `main`
2. **Make your changes** with clear commit messages
3. **Open a pull request** with appropriate labels
4. **Test the PR image** using the Docker command from the auto-comment
5. **Address review feedback** if needed
6. **Merge when approved** → Draft release is automatically updated

### Testing Images

#### PR Images
Every PR gets a unique Docker image for testing:
```bash
# Example: PR #123 with commit abc1234
docker pull ghcr.io/hylo-lang/hylo-dev-toolchain:pr-123-abc1234
docker run --rm -it ghcr.io/hylo-lang/hylo-dev-toolchain:pr-123-abc1234
```

#### Draft Release Images
Test upcoming releases before they're published:
```bash
docker pull ghcr.io/hylo-lang/hylo-dev-toolchain:latest-draft
docker run --rm -it ghcr.io/hylo-lang/hylo-dev-toolchain:latest-draft
```

## Release Management

### Automatic Release Process

1. **Merge PRs** with appropriate labels → Draft release is updated
2. **Review the draft** at [Releases page](../../releases)
3. **Test the draft image** if needed
4. **Publish the release** → Final Docker image is built and tagged

### Manual Release Process

1. Go to [Releases page](../../releases)
2. Click "Draft a new release"
3. Choose tag (e.g., `v1.2.0`) and fill release notes
4. Publish → Docker image is automatically built

### Available Docker Tags

- `latest` → Latest stable release
- `v1.0.0` → Specific version tags
- `latest-draft` → Current draft release (for testing)
- `pr-123-abc1234` → PR-specific images (temporary)

## Image Cleanup Policy

- **PR images**: Deleted when PR is closed/merged
- **Draft images**: Replaced when new draft is created
- **Old experimental images**: Cleaned up weekly (>7 days old)
- **Release images**: Never automatically deleted

## Automatic Labeling

Some labels are automatically applied based on changed files:
- `documentation` → `.md` files, `docs/` folder
- `ci` → `.github/workflows/` files
- `dependencies` → `package.json`, `requirements.txt`, etc.
- `build` → `Dockerfile*`, `make-pkgconfig.sh`
