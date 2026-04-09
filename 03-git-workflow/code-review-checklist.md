# Code Review Checklist

Use this checklist when reviewing pull requests. Not every item applies to every PR — use judgment.

## Correctness

- [ ] The code does what the PR description says it does
- [ ] Edge cases are handled (null, empty, boundary values)
- [ ] Error handling is appropriate (not swallowed, not over-caught)
- [ ] No off-by-one errors, race conditions, or resource leaks

## Security

- [ ] No hardcoded secrets, API keys, or passwords
- [ ] User input is validated and sanitized
- [ ] No SQL injection, XSS, or command injection vectors
- [ ] Auth/authz checks are in place where needed
- [ ] Sensitive data is not logged

## Design

- [ ] Changes are in the right layer (domain vs. infra vs. API)
- [ ] No unnecessary coupling between modules
- [ ] Public API surface is minimal — no premature exposure
- [ ] Naming is clear and consistent with the codebase

## Quality

- [ ] No dead code, commented-out code, or TODOs without an issue link
- [ ] No duplication that should be factored out
- [ ] Functions are focused (single responsibility)
- [ ] Code is readable without needing the PR description to understand it

## Tests

- [ ] New code has tests
- [ ] Tests are meaningful (not just asserting `true`)
- [ ] Edge cases are tested
- [ ] No `.only`, `@skip`, or `@Disabled` left behind
- [ ] Tests pass locally and in CI

## Documentation

- [ ] Public APIs are documented
- [ ] Breaking changes are noted
- [ ] CHANGELOG is updated if user-facing

## Pragmatic checks

- [ ] Is this the simplest solution that works?
- [ ] Would I understand this code in 6 months?
- [ ] Does the PR do one thing? (If not, should it be split?)

---

*Tip: for solo maintainers, do a self-review using this checklist before merging. Read your own diff as if someone else wrote it.*
