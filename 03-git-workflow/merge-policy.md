# Merge Policy

## Requirements before merge

- [ ] CI is green (all checks pass)
- [ ] At least <!-- REPLACE: 1 --> approving review
- [ ] No unresolved review comments
- [ ] Branch is up to date with `main`
- [ ] PR description is filled in

## Merge strategy

| Branch type | Merge method | Why |
|-------------|-------------|-----|
| Feature | **Squash merge** | Clean history — one commit per feature |
| Bugfix | **Squash merge** | Same — keeps `main` log readable |
| Release | **Merge commit** | Preserves the full release branch history |
| Hotfix | **Squash merge** | Fast, minimal noise |

### Squash merge rules

- Use the PR title as the commit message
- Follow [commit conventions](commit-conventions.md)
- Delete the source branch after merge

## Who can merge

<!-- REPLACE: describe who has merge rights -->

| Role | Can merge? |
|------|-----------|
| Maintainer | Yes |
| Contributor | No — maintainer merges after approval |

## Stale PRs

PRs with no activity for **<!-- REPLACE: 14 -->** days will be labeled `stale`. After **<!-- REPLACE: 7 -->** more days, they will be closed with a comment inviting the author to reopen.

## Conflicts

- The PR author is responsible for resolving merge conflicts
- Prefer rebasing on `main` over merge commits in feature branches
