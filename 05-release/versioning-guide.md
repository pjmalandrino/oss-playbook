# Versioning Guide

This project follows [Semantic Versioning](https://semver.org/) (SemVer).

## Format

```
MAJOR.MINOR.PATCH
```

## When to bump what

| Change type | Bump | Example |
|-------------|------|---------|
| Breaking API change | **MAJOR** | Remove endpoint, rename field, change auth flow |
| New feature (backward-compatible) | **MINOR** | Add endpoint, add optional field, new UI page |
| Bug fix (backward-compatible) | **PATCH** | Fix crash, correct calculation, typo in response |
| Documentation only | **PATCH** | Fix README, update API docs |
| Dependency update (no behavior change) | **PATCH** | Bump library version |
| Dependency update (breaking) | Depends | If it changes your public API → MAJOR; otherwise PATCH |
| Internal refactor (no behavior change) | **PATCH** | Rename internal class, restructure modules |

## Pre-release versions

For versions before 1.0.0:

- `0.x.y` — anything can change at any time
- Breaking changes bump MINOR: `0.1.0` → `0.2.0`
- Bug fixes bump PATCH: `0.1.0` → `0.1.1`

## When to go 1.0.0

You're ready for 1.0.0 when:

- [ ] The public API is stable and documented
- [ ] You have users depending on the current behavior
- [ ] Breaking changes would cause real pain
- [ ] You're comfortable committing to backward compatibility

## Version in code

Update the version in these files before each release:

<!-- REPLACE: list the files where version appears -->

```
- pyproject.toml / package.json / build.gradle
- __version__.py / version.ts
- Dockerfile labels (if any)
- API docs (OpenAPI spec)
```

## Git tags

- Always tag releases: `v1.2.3`
- Annotated tags preferred: `git tag -a v1.2.3 -m "Release v1.2.3"`
- Never move or delete a published tag
