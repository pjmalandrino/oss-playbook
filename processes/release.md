# Release Process

> **When to use**: Feature freeze reached, ready to ship a new version.

## Steps

### 1. Create the release branch

```bash
git checkout main && git pull
git checkout -b release/X.Y.Z
```

### 2. On the release branch (only)

- Bug fixes only (no new features)
- Move `[Unreleased]` to `[X.Y.Z] - YYYY-MM-DD` in `CHANGELOG.md`
- Update `version` in `package.json`

### 3. Merge to main via PR

The PR triggers the **[release gate](../ci-cd/release-gate.md)** which runs:
- Lint backend + frontend
- Unit tests backend + frontend
- Docker build (smoke)
- Full E2E (@smoke + @regression + @e2e)

### 4. Tag on main

```bash
git checkout main && git pull
git tag vX.Y.Z
git push origin vX.Y.Z
```

### 5. Automatic release (GitHub Actions)

Pushing the tag triggers the release workflow ([details](../ci-cd/release.md)):

- Docker **multi-arch** build (linux/amd64 + linux/arm64)
- 2 targets in matrix: `remote` and `local`
- Automatically generated tags:
  - `X.Y.Z-remote`, `X.Y-remote`, `latest-remote`
  - `X.Y.Z-local`, `X.Y-local`, `latest-local`
- Push to `ghcr.io`
- Cache via GitHub Actions cache

## Versioning

- **Semantic Versioning**: MAJOR.MINOR.PATCH
- **Source of truth**: Git tag `vX.Y.Z`
- `package.json` must match the current release
- Version auto-injected:
  - Frontend: Vite `__APP_VERSION__` (via `VITE_APP_VERSION`)
  - Backend: `APP_VERSION` environment variable

## Red flags

- New feature commit on the release branch — only bug fixes allowed
- `CHANGELOG.md` still shows `[Unreleased]` at tag time — must be updated
- `package.json` version doesn't match the tag — version mismatch in production
- Tagging before the release gate is green — audit bypassed
- Force-pushing to the release branch — history rewrite breaks audit trail

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "Let's sneak this feature into the release" | Feature freeze exists to stabilize. New features need a new cycle |
| "The audit is mostly green, let's skip the remaining MAJs" | MAJs compound. Shipping known issues erodes quality standards |
| "We can tag now and fix the CHANGELOG later" | The CHANGELOG is the user-facing contract. "Later" never comes |
| "Multi-arch build is slow, let's skip ARM" | ARM users exist (local dev on Apple Silicon). Ship both or don't ship |

## Verification

- [ ] Release branch contains only bug fixes since branch creation
- [ ] `CHANGELOG.md` has the correct version and date
- [ ] `package.json` version matches `X.Y.Z`
- [ ] Release gate verdict is **GO** or **GO CONDITIONAL** with accepted MAJs
- [ ] Tag is on `main` (not on the release branch)
- [ ] Docker images are published on ghcr.io (check GitHub Actions)
- [ ] `latest-remote` and `latest-local` point to the new version
