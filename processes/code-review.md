# Code Review

> **When to use**: Every PR, before merge. No exceptions.

## Before submitting the PR

- [ ] Branch named correctly (`feature/`, `fix/`, `hotfix/`)
- [ ] Commits follow Conventional Commits
- [ ] Tests added/updated
- [ ] All tests pass
- [ ] Linting passes (ruff / eslint)
- [ ] `CHANGELOG.md` updated
- [ ] Documentation updated if needed
- [ ] No secrets committed

## Review axes

### Functional
- Feature does what is requested
- Edge cases are handled
- Behavior is consistent with the rest of the app

### Architecture
- Respects hexagonal architecture (domain depends on nothing, adapters implement ports)
- No duplication of business logic
- DTOs are properly separated from domain models
- Errors are handled cleanly (no silent `pass`)
- New code is in the right layer (see [architecture](../README.md#backend-architecture-hexagonal--ports--adapters))

### Security
- DOMPurify for all dynamic HTML (frontend)
- Input validation at API level (Pydantic + FastAPI Depends)
- No hardcoded secrets
- Uploads are validated (MIME type, size)
- See [Security checklist](../references/security-checklist.md) for full list

### Performance
- No N+1 queries
- Heavy operations are async
- Rate limiting is in place
- See [Performance checklist](../references/performance-checklist.md) for full list

### Tests
- New code has corresponding tests
- Tests are deterministic (no flaky tests)
- Test names describe behavior, not implementation
- See [Testing patterns](../references/testing-patterns.md) for conventions

## PR template

Every PR uses the standardized template with:
- Type (Feature, Bug fix, Hotfix, Documentation, Refactoring, CI/CD)
- Summary in 1-3 sentences
- Related issues
- Validation checklist
- Screenshots/evidence if applicable

## Red flags

- PR changes more than 400 lines — should be split into smaller PRs
- No tests added for new behavior — tests are not optional
- Domain layer imports from `api/`, `persistence/`, or `infra/` — hexagonal violation
- `# TODO` without issue reference — will never be addressed
- `console.log` left in frontend code — use `console.warn`/`console.error` only
- Pydantic schema uses snake_case — frontend DTOs must be camelCase
- Silent `except: pass` — errors must be handled explicitly

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "It's a small change, review is overkill" | Small changes can introduce subtle bugs. Review catches what tests miss |
| "I wrote the tests, that's enough" | Tests verify your assumptions. Review challenges your assumptions |
| "We're in a hurry, just merge it" | Rushing creates tech debt. A 15-min review saves hours of debugging |
| "It's just refactoring, nothing functional changed" | Refactoring can break contracts. Review ensures behavior is preserved |
| "The CI is green" | CI checks syntax and known test cases. Review checks design and intent |

## Verification

- [ ] All checklist items above are checked
- [ ] CI pipeline is green
- [ ] At least 1 approval from a reviewer
- [ ] No unresolved comments
- [ ] PR description accurately reflects the changes
