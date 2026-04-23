---
description: Start implementation on a Docling Studio issue — validates metadata + design doc, creates the working branch, links it back to the issue
argument-hint: <issue-number> [--no-design] [--force] [--from <base>]
allowed-tools: Bash(gh auth status:*), Bash(gh issue view:*), Bash(gh issue list:*), Bash(gh issue comment:*), Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(git pull:*), Bash(git fetch:*), Bash(git remote get-url:*), Bash(git rev-parse:*), Bash(git config:*), Bash(ls:*), Read, Grep, Glob
---

# /resolve — Start implementing an issue

Bootstrap the implementation phase on a triaged + designed Docling Studio
issue: validate the issue is actually ready for coding, check the working
tree is clean, create the correctly-named branch off `main`, and post a
pointer comment on the issue. **No code is written by this command.**

## Hard constraints (non-negotiable)

1. **Issue number required.** `/resolve` with no arg lists open issues and
   asks — it never picks one.
2. **Issue is OPEN**, with **exactly one** type label (`bug`,
   `enhancement`, `documentation`, `test`, `chore` — no `feature` label on
   the repo; see `processes/issue-creation.md`), a **milestone**, and a
   canonical `[TAG]` prefix in the title.
3. **Design doc required** for `enhancement` issues OR any `[FEATURE]`-
   prefixed title: a file matching `docs/design/<n>-*.md` must exist with
   status `Accepted`. Skip this check with `--no-design` only for
   deliberate exceptions (trivial refactors, obvious fixes).
4. **cwd must be a git checkout of `scub-france/Docling-Studio`.** Verify
   via `git remote get-url origin`. Refuse otherwise — do not try to
   guess the right path.
5. **Working tree must be clean.** `git status --porcelain` must be empty.
   Refuse if dirty — do not auto-stash.
6. **Branch must not already exist locally** (same name) unless `--force`.
   If it exists, suggest `git checkout <branch>` to resume instead.
7. **Never push the branch automatically.** Creation is local; pushing is
   the author's call (typically via `git push -u origin <branch>` when
   ready for a PR).

## Arguments

User invocation: `/resolve $ARGUMENTS`

Expected shape:

```
/resolve <issue-number> [--no-design] [--force] [--from <base>]
```

Flags:

- `--no-design` — skip the design-doc precondition (use sparingly; records
  an explicit exception in the issue comment).
- `--force` — recreate the branch even if one with the same name exists.
- `--from <base>` — base branch (default: `main`). Use `release/X.Y.Z` for
  hotfix preparation, but prefer the dedicated [Hotfix](../../processes/hotfix.md)
  process for production-critical bugs.

Examples:

- `/resolve 142`
- `/resolve 87 --no-design` (documentation or trivial chore)
- `/resolve 310 --from release/0.7.0`

## Workflow

1. **Auth & environment checks.**
   - `gh auth status` — stop and instruct `gh auth login` if not signed in.
   - `git rev-parse --is-inside-work-tree` — must be `true`.
   - `git remote get-url origin` — must match
     `scub-france/Docling-Studio` (HTTPS or SSH).

2. **Fetch the issue.**
   ```bash
   gh issue view <n> --repo scub-france/Docling-Studio \
     --json number,title,body,state,labels,milestone,url
   ```

3. **Validate metadata.**
   - `state` must be `OPEN`.
   - Exactly one label from the allowed set (`bug`, `enhancement`,
     `documentation`, `test`, `chore`). Refuse if zero or multiple.
   - `milestone` must be non-null.
   - Title must start with a canonical `[TAG]` prefix (`[BUG]`,
     `[FEATURE]`, `[ENHANCEMENT]`, `[DOC]`, `[TEST]`, `[CHORE]`). Tolerate
     legacy `[INFRA]` / `[FIX]` with a warning.

4. **Check for the design doc.**
   - Glob `docs/design/<n>-*.md`. Pick the first match.
   - If the type resolves to `enhancement` OR the title tag is `[FEATURE]`
     / `[ENHANCEMENT]`, **a design doc is required** unless `--no-design`
     was passed.
   - Read the doc header and confirm `**Status**: Accepted`. Warn (don't
     refuse) on `Draft` / `Review` — imply the user may want to run
     `/conception` first or flip the status.

5. **Check the working tree.**
   - `git status --porcelain` must be empty. Refuse if dirty; tell the
     user to commit or stash (do not stash automatically).

6. **Sync the base branch.**
   - `git fetch origin`
   - `git checkout <base>` (default `main`)
   - `git pull --ff-only origin <base>` — refuse on non-fast-forward; the
     base should be clean.

7. **Compute the branch name.** `<prefix>/<n>-<slug>`, with:

   | Issue type / tag | Branch prefix |
   |---|---|
   | `bug` / `[BUG]` / `[FIX]` | `fix` |
   | `enhancement` + `[FEATURE]` | `feature` |
   | `enhancement` + `[ENHANCEMENT]` | `feature` |
   | `documentation` / `[DOC]` | `docs` |
   | `test` / `[TEST]` | `test` |
   | `chore` / `[CHORE]` / `[INFRA]` | `chore` |

   `<slug>` is derived from the issue title **after stripping the `[TAG]`
   prefix**: 4–6 meaningful words, kebab-case, ASCII-only, stop-words
   dropped. Example: issue #142 "[FEATURE] Expose batch progress via SSE"
   → `feature/142-expose-batch-progress-sse`.

8. **Refuse if the branch exists** (`git branch --list <branch>` non-empty)
   unless `--force`. With `--force`, delete the old local branch first and
   re-announce clearly.

9. **Create and checkout the branch.**
   ```bash
   git checkout -b <branch>
   ```

10. **Post a pointer comment on the issue.** Ask first; default yes.
    ```bash
    gh issue comment <n> --repo scub-france/Docling-Studio \
      --body "Working this issue on branch \`<branch>\` (base: \`<base>\`)."
    ```
    If `--no-design` was used, the comment must explicitly say so:
    `"Working this issue on branch \`<branch>\` (base: \`<base>\`, no design doc — exception)."`

11. **Print the handoff summary.** Include, in this order:
    - Issue URL + type + milestone + title.
    - Design doc path (or `— none (exception)`).
    - Branch name + base.
    - Scope labels (`backend`, `frontend`, `infra`, `search`, etc.) — drives
      which validation pipeline to run.
    - The validation pipeline to run **before every commit** (backend
      Ruff+pytest, frontend ESLint+vue-tsc+vitest, or both — see
      `claude-code/workflows.md` in the playbook).
    - A one-line nudge: "Next: open/fill in the design doc's §9 Testing
      strategy if not already done, then write a failing test first."

## Refusals and recovery

| Situation | Behavior |
|---|---|
| No arg | Show the 10 most recent open issues with type+milestone, ask |
| Issue closed / missing | Stop. Suggest re-opening or `/create-issue` |
| Missing type label | Stop. Send the user to triage |
| Missing milestone | Stop. Send the user to `/create-issue` or triage |
| Bad `[TAG]` prefix | Stop. Send the user to `/create-issue` to rewrite the title |
| Required design doc missing | Stop. Suggest `/conception <n>` (or `--no-design` if truly trivial) |
| Design doc status ≠ `Accepted` | Warn, offer to proceed; default: stop |
| Dirty working tree | Stop. Tell user to commit or stash — do not stash automatically |
| Not in a Docling-Studio checkout | Stop. Do not guess the right path |
| Branch already exists | Stop. Suggest `git checkout <branch>` to resume (`--force` to recreate) |
| Base branch non-FF on pull | Stop. User must reconcile `main` first |
| `gh` not authenticated | Stop. `gh auth login` |

## Out of scope for this command

- Writing the implementation.
- Running lint / tests / build (use the validation pipeline from
  `claude-code/workflows.md`).
- Opening the PR (do it via `gh pr create` once the branch has commits
  and the validation pipeline is green).
- Closing the issue (`Closes #<n>` in the PR body or merge commit handles
  this automatically).

## Why this command exists

See `processes/resolve.md` in the playbook for the full rationale, red
flags, and verification checklist. In short: it prevents starting work on
an under-triaged issue, eliminates off-convention branch names, and keeps
the issue ↔ branch ↔ design-doc graph consistent.
