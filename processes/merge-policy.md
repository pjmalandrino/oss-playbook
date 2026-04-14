# Merge Policy

> **When to use**: PR is ready to be merged into `main` or `release/*`.

## Merge conditions

1. **Green CI** — All jobs pass (backend, frontend, e2e if applicable)
2. **Approved review** — At least 1 approval
3. **No conflicts** — Branch is up to date with base
4. **PR checklist complete** — All items checked

## Merge strategy

- **Squash merge** for feature branches -> `main`
- **Merge commit** for release branches -> `main` (preserves history)
- **Rebase** for hotfix branches

## Protected branches

| Branch | Rules |
|--------|-------|
| `main` | PR required, CI required, review required |
| `release/*` | PR required, CI required + full E2E |

## Red flags

- Merging with unresolved review comments — all conversations must be resolved
- Merging with failing CI "because it's a flaky test" — fix the flaky test first
- Force-merging by bypassing branch protection — never acceptable
- Merge commit on a feature branch — should be squash merge

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "The CI failure is unrelated to my change" | If it's truly unrelated, fix it in a separate PR first |
| "I'll address the review comment in a follow-up PR" | Follow-up PRs for review comments have a 90% abandon rate |
| "It's Friday afternoon, let's just merge" | Friday merges become Monday incidents. Don't deploy on Friday |

## Verification

- [ ] CI is fully green (no skipped or ignored failures)
- [ ] All review comments are resolved
- [ ] Correct merge strategy used (squash/merge commit/rebase)
- [ ] Branch is deleted after merge
