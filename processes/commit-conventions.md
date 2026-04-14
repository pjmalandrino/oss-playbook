# Commit Conventions

> **When to use**: Every single commit, no exceptions.

Convention: **Conventional Commits**

## Format

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

## Types

| Type | Usage |
|------|-------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting (no logic change) |
| `refactor` | Code restructuring without functional change |
| `test` | Adding or modifying tests |
| `chore` | Dependencies, tooling, config |
| `perf` | Performance improvement |
| `ci` | CI/CD changes |

## Scopes used

`api`, `domain`, `persistence`, `infra`, `upload`, `analysis`, `chunking`, `bbox`, `e2e`, `docker`, `ci`

## Rules

1. Imperative mood, lowercase, no trailing period, max 72 characters
2. Body explains **why**, not **what**
3. Breaking changes: `BREAKING CHANGE:` footer or `!` after type
4. Issue references: `Closes #123`, `Fixes #456`

## Examples

```
feat(analysis): add batch progress tracking

Track conversion progress per page and emit SSE events
to update the frontend in real-time.

Closes #87
```

```
fix(api): prevent duplicate upload when file already exists

The upload endpoint was creating a new entry even when a file
with the same hash existed. Now returns 409 Conflict.
```

```
refactor(persistence)!: migrate from JSON files to aiosqlite

BREAKING CHANGE: data directory structure has changed,
existing uploads require migration.
```

## Red flags

- Commit message says "fix stuff" or "WIP" — rewrite before pushing
- Body describes **what** changed instead of **why** — the diff already shows what
- Multiple unrelated changes in a single commit — split into atomic commits
- No scope on a commit that clearly touches one module — add the scope
- Breaking change without `!` or `BREAKING CHANGE:` footer — will be missed in changelog

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "It's just a small fix, no need for format" | Small fixes become invisible in git log without proper type/scope |
| "I'll clean up the commits later" | You won't. Interactive rebase on shared branches causes conflicts |
| "Nobody reads commit messages" | CI changelog generation, git bisect, and blame all depend on them |
| "The PR description explains everything" | PRs disappear after merge. Commit messages are permanent |

## Verification

- [ ] `git log --oneline -5` shows consistent `type(scope): description` format
- [ ] No commit exceeds 72 characters in the first line
- [ ] Breaking changes are explicitly marked
- [ ] Every commit compiles and passes tests independently (no "fix previous commit")
