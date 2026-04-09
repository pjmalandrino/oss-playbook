# Release Process

Step-by-step checklist for shipping a new version.

## Pre-release

- [ ] All planned issues for this release are closed
- [ ] Release branch created: `release/<version>`
- [ ] Version bumped in code/config: <!-- REPLACE: e.g. pyproject.toml, package.json -->
- [ ] CHANGELOG.md updated (move "Unreleased" items to new version section)
- [ ] Release audit passed (see [04-quality/release-audit/](../04-quality/release-audit/))
- [ ] All CI checks green on release branch
- [ ] No unresolved CRITICAL findings
- [ ] Dependencies audited (`npm audit`, `pip-audit`, etc.)

## Build and tag

- [ ] Merge release branch to `main`
- [ ] Tag the merge commit:

```bash
git tag -a v<version> -m "Release v<version>"
git push origin v<version>
```

- [ ] Verify the CI/CD pipeline triggers on the tag
- [ ] Build artifacts are produced (Docker image, package, binary, etc.)

## Deploy

- [ ] Follow [deployment-checklist.md](deployment-checklist.md)
- [ ] Smoke test in production
- [ ] Verify health checks / monitoring dashboards

## Post-release

- [ ] Create GitHub Release with release notes
- [ ] Close the milestone (if using milestones)
- [ ] Announce the release: <!-- REPLACE: e.g. Discord, Twitter, mailing list -->
- [ ] Merge `main` back to `develop` (if using Gitflow)
- [ ] Delete the release branch

## If something goes wrong

Follow the [rollback-playbook.md](rollback-playbook.md).
