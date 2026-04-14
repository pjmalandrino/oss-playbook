# Hotfix Process

> **When to use**: Critical production bug requiring an immediate patch, without waiting for the normal release cycle.

## Steps

### 1. Create the hotfix branch from the tag

```bash
git checkout vX.Y.Z
git checkout -b hotfix/X.Y.Z+1
```

### 2. Fix and commit

```bash
# Apply the fix
git commit -m "fix(scope): description of the critical fix"
```

### 3. PR to main

- CI validates the fix
- Expedited review (1 approval minimum)

### 4. Tag after merge

```bash
git checkout main && git pull
git tag vX.Y.Z+1
git push origin vX.Y.Z+1
```

The tag automatically triggers the Docker build and push.

## Important notes

- A hotfix contains **only** the fix, nothing else
- Update `CHANGELOG.md` with the hotfix section
- Verify E2E tests pass before tagging

## Red flags

- Hotfix branch contains unrelated changes — must be ONLY the fix
- No test covering the bug being fixed — the bug will come back
- Hotfix created from `main` instead of from the tag — may include unreleased code
- Multiple hotfixes stacked without releasing — use a proper release cycle instead

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "While we're at it, let's include this other fix" | Scope creep in hotfixes is how you break production twice |
| "It's obvious it works, we don't need a test" | The bug exists because there was no test. Add one |
| "Review slows us down in an emergency" | 1 quick review catches the 2nd bug you introduce while stressed |
| "Let's skip the E2E, it's just a small fix" | Small fixes can have large blast radius. E2E is the safety net |

## Verification

- [ ] Hotfix branch created from the correct release tag
- [ ] Only the fix is in the branch (nothing else)
- [ ] Test added that reproduces the original bug
- [ ] CI is green
- [ ] E2E @smoke passes
- [ ] Tag is on `main` after merge
- [ ] Docker images are published
