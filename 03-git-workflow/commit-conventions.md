# Commit Conventions

## Format

This project follows [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type

| Type | When to use |
|------|------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `chore` | Build, CI, tooling, dependencies |
| `perf` | Performance improvement |

### Scope (optional)

The part of the codebase affected: `auth`, `api`, `ui`, `db`, etc.

### Subject

- Imperative mood, present tense: "add" not "added" or "adds"
- No period at the end
- Max 72 characters

### Body (optional)

Explain **why**, not what. The diff shows what changed.

### Footer (optional)

- `BREAKING CHANGE: <description>` for breaking changes
- `Closes #123` to auto-close issues
- `Co-Authored-By: Name <email>` for pair work

## Examples

```
feat(auth): add OAuth2 login with GitHub

Users can now log in using their GitHub account.
The OAuth flow uses PKCE for security.

Closes #42
```

```
fix(api): return 404 instead of 500 for missing documents
```

```
chore: upgrade ruff to 0.4.0
```

## Enforcement

<!-- REPLACE: describe how you enforce this (commitlint, CI check, honor system) -->
