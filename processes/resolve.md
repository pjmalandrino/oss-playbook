# Issue Resolution (Implementation Bootstrap)

> **When to use**: You're about to start coding on an issue. Runs **after** [Issue Creation](issue-creation.md), [Triage](issue-triage.md), and (for non-trivial work) [Conception](conception.md). Runs **before** any commit.

## The command

Use the Claude Code slash command `/resolve` (defined in
[`.claude/commands/resolve.md`](../.claude/commands/resolve.md)) — never
create the working branch by hand. The command enforces the full
precondition chain, keeps branch names consistent with
[Commit Conventions](commit-conventions.md), and links the branch back to
the issue.

Shape:

```
/resolve <issue-number> [--no-design] [--force] [--from <base>]
```

## Mandatory preconditions

The command **refuses** if any is missing:

| Precondition | Why |
|---|---|
| Issue is OPEN | Working on a closed issue hides the work from the backlog |
| Exactly one type label (`bug`/`enhancement`/`documentation`/`test`/`chore`) | Determines the branch prefix and the validation pipeline |
| Milestone set | Ties the work to a release |
| Canonical `[TAG]` title prefix | Enables the slug derivation and keeps branch names scannable |
| Design doc `Accepted` for `enhancement` / `[FEATURE]` / `[ENHANCEMENT]` | Prevents "design-in-the-PR" drift |
| Working tree clean | Avoids mixing unrelated work onto the new branch |
| cwd is a `scub-france/Docling-Studio` checkout | Don't guess the repo |
| Branch doesn't already exist locally | Resume via `git checkout`; recreate only with `--force` |

## Branch naming

`<prefix>/<issue-number>-<slug>`, where `<prefix>` maps from the issue type
and title tag:

| Issue type | Title tag | Branch prefix |
|---|---|---|
| `bug` | `[BUG]` / `[FIX]` | `fix` |
| `enhancement` | `[FEATURE]` | `feature` |
| `enhancement` | `[ENHANCEMENT]` | `feature` |
| `documentation` | `[DOC]` | `docs` |
| `test` | `[TEST]` | `test` |
| `chore` | `[CHORE]` / `[INFRA]` | `chore` |

`<slug>` is derived from the issue title **after stripping the `[TAG]`
prefix**: 4–6 meaningful words, kebab-case, ASCII-only. Example:

- Issue #142 "[FEATURE] Expose batch progress via SSE"
  → branch `feature/142-expose-batch-progress-sse`

**Hotfix exception.** Critical production bugs use the [Hotfix](hotfix.md)
process, not `/resolve`. Pass `--from release/X.Y.Z` only when you
explicitly need to branch off a release line and you've decided `/resolve`
is still the right tool.

## Workflow

1. Triage + conception done (see preconditions).
2. `git fetch && git status` — confirm working tree is clean.
3. **Run `/resolve <n>`.** The command syncs `main`, creates the branch,
   and posts the pointer comment on the issue.
4. Open the design doc (if any) and start with §9 Testing strategy — write
   the failing test before the production code.
5. Iterate: implement, run the relevant validation pipeline (see
   [Claude Code Workflows](../claude-code/workflows.md)), commit with
   [Conventional Commits](commit-conventions.md).
6. When ready: `git push -u origin <branch>` and open a PR (`gh pr create`)
   with `Closes #<n>` in the body. See [Code Review](code-review.md) and
   [Merge Policy](merge-policy.md) for what happens next.

## Red flags

- Branch name doesn't match `<prefix>/<n>-<slug>` — the command was bypassed
  or the issue had no `[TAG]` prefix
- Branch created with no corresponding open issue — ghost work
- Work on a `feature/*` branch with no accepted design doc — `--no-design`
  was abused, or the conception step was skipped
- Same branch recreated multiple times with `--force` — likely masking a
  reconciliation problem on `main`
- Branch pushed directly without PR — breaks code review and CI gates
- Issue has no comment pointing at the working branch — traceability broken

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "It's a two-line fix, I don't need a branch" | Two-line fixes still need PR review, CI, and a changelog entry. The branch is the unit of all three |
| "I'll link the issue to the PR, the branch name doesn't matter" | Branch names show up in CI logs, deploy notifications, and `git log` — they're public-facing |
| "I'll skip the design doc, I know what to do" | `/resolve --no-design` exists for a reason. Use it deliberately, not by default — and leave the exception comment on the issue |
| "The working tree is only slightly dirty" | Slightly-dirty trees leak into the new branch as surprise commits nobody intended |
| "I'll just branch off my last feature branch, it's faster" | Stacked branches break `main`-based CI gates and make reverts painful. Always branch off `main` unless the hotfix process says otherwise |

## Verification

- [ ] Every open branch on the remote (`git branch -r`) matches `<prefix>/<n>-<slug>` for an open issue
- [ ] No branch named only with a number (`142`) or prose (`new-sse-progress`)
- [ ] Every implementation PR has a `Closes #<n>` footer
- [ ] Every `feature/*` branch has a corresponding `docs/design/<n>-*.md` unless the PR description carries an explicit `--no-design` exception note
- [ ] `git log --all --format='%h %s' | grep -iE '\bwip\b|\bfix me\b|\btodo\b'` returns empty for merged commits
- [ ] Issue timeline shows the branch pointer comment within a few minutes of branch creation
