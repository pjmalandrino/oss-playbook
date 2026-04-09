# Branching Strategy

## Choose your model

### Trunk-Based Development

Best for: solo maintainers, small teams, CI/CD-heavy projects.

```
main ─────●────●────●────●────●─────
           \       /      \      /
            feat-A          feat-B
```

- Everyone works off `main`
- Short-lived feature branches (< 2 days)
- No `develop` branch
- Deploy from `main` after every merge

### GitHub Flow

Best for: OSS projects with external contributors.

```
main ─────●──────────●──────────●─────
           \        /  \        /
            feat-A      fix-B
            (PR)        (PR)
```

- `main` is always deployable
- Feature branches + pull requests
- Merged via squash or merge commit
- Release = tag on `main`

### Gitflow

Best for: projects with scheduled releases and long support cycles.

```
main ─────●──────────────●────────────
           \            / \
develop ────●────●────●    release/1.0
             \      /
              feat-A
```

- `main` = production
- `develop` = integration
- `release/*` branches for stabilization
- `hotfix/*` branches for urgent prod fixes

## Our strategy

<!-- REPLACE: Pick one and describe any adaptations -->

**Model**: <!-- REPLACE: Trunk-Based / GitHub Flow / Gitflow -->

**Branch naming**:

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feat/<description>` | `feat/add-auth` |
| Bug fix | `fix/<description>` | `fix/null-pointer-upload` |
| Release | `release/<version>` | `release/2.1.0` |
| Hotfix | `hotfix/<description>` | `hotfix/cors-wildcard` |

**Protected branches**:

- `main` — requires PR, CI pass <!-- REPLACE: adapt -->
