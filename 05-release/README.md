# 05 — Release

Everything you need to ship a version with confidence.

## What's here

| Template | Purpose |
|----------|---------|
| [release-process.md](release-process.md) | Step-by-step release checklist |
| [versioning-guide.md](versioning-guide.md) | When to bump major, minor, or patch |
| [deployment-checklist.md](deployment-checklist.md) | Pre and post-deploy verification |
| [rollback-playbook.md](rollback-playbook.md) | What to do when a release goes wrong |

## Typical flow

```
1. Feature freeze → create release branch
2. Run release audit (04-quality/)
3. Fix blockers
4. Follow release-process.md
5. Tag, build, deploy (deployment-checklist.md)
6. If something goes wrong → rollback-playbook.md
```
