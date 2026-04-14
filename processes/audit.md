# Quality Audit

> **When to use**: Before every merge of a `release/*` branch into `main`. Non-negotiable.

Complete audit framework with **12 axes**, each being a weighted checklist.
The audit runs before every merge of a `release/*` branch into `main`.

## The 12 audit axes

| # | Axis | Focus |
|---|------|-------|
| 01 | **Hexagonal Architecture** | Domain isolation, ports & adapters, no layer leaks |
| 02 | **DDD** | Bounded contexts, value objects, ubiquitous language |
| 03 | **Clean Code** | Naming, file size, readability |
| 04 | **KISS** | Simplicity, no over-engineering |
| 05 | **DRY** | Duplication detection, magic numbers |
| 06 | **SOLID** | The 5 SOLID principles |
| 07 | **Decoupling** | Frontend/backend, contracts, ports & adapters |
| 08 | **Security** | OWASP, secrets, injection, CORS |
| 09 | **Tests** | Coverage, quality, determinism, E2E |
| 10 | **CI / Build** | Pipeline, Docker health checks |
| 11 | **Documentation** | CHANGELOG, TODOs with issue refs |
| 12 | **Performance** | N+1 queries, async I/O, memory |

## Invocation modes

| Mode | When | Output |
|------|------|--------|
| **Full Release Audit** | Before merge release -> main | 12 reports + summary |
| **Single Audit** | Targeted check on one axis | 1 report |
| **Re-Audit** | After fixing CRIT/MAJ | Updated report |
| **Automated Checks** | Quick pre-audit validation | PASS/WARN/FAIL per axis |

## Severity levels

| Tag | Meaning | Impact |
|-----|---------|--------|
| `[CRIT]` | Blocking (security, data corruption, major architecture violation) | **Absolute NO-GO** |
| `[MAJ]` | Significant (strong coupling, critical test gap) | Must fix before release |
| `[MIN]` | Quality gap (naming, file size) | Recommended improvement |
| `[INFO]` | Observation / improvement track | Informational |

## Scoring

### Per-axis calculation

```
score = (sum of passing item weights / total weights) * 100
```

### Decision thresholds

| Score | Verdict | Condition |
|-------|---------|-----------|
| >= 80 | **GO** | All checks pass |
| 60-79 | **GO CONDITIONAL** | 0 CRIT + remediation plan for MAJ |
| < 60 | **NO-GO** | Mandatory corrections |

**Absolute rule**: any `[CRIT]` = **NO-GO**, regardless of score.

## Automated checks (script)

Bash script that runs automatable verifications across all 12 axes:

```bash
# What the script checks:
- Domain layer: no forbidden imports (FastAPI, SQLite, etc.)
- File size: warning > 300 lines
- Magic numbers: hardcoded value detection
- Secrets: pattern scanning (password, token, secret)
- SQL injection: dangerous pattern detection
- CORS: configuration present
- Tests: test files exist for each module
- Documentation: CHANGELOG.md and README.md present
- N+1 queries: suspicious pattern detection

# Output
- PASS (green) / WARN (yellow) / FAIL (red) per check
- Exit code 1 if any FAIL
```

## CI/CD integration (Release Gate)

The release gate runs the audit in 4 phases:

### Phase 1 — Parallel checks (blocking)

- Lint & type-check (ruff, ESLint, vue-tsc)
- Unit tests (pytest, Vitest)
- **Dependency audit** (`pip-audit` + `npm audit`) — blocking on CRITICAL
- **Audit checks** (automated script) — blocking

### Phase 2 — Docker (blocking)

- Docker build (targets remote + local)
- Smoke test (`/api/health`)
- **Trivy image scan** — blocking on CRITICAL, warning on HIGH
- Image size check (delta vs previous)

### Phase 3 — E2E

- E2E API (full scope: @smoke + @regression + @e2e)
- E2E UI (@critical only)

### Phase 4 — Verdict

Automatic comment on the PR with verdict:

| Verdict | Meaning |
|---------|---------|
| **GO** | All checks pass |
| **GO CONDITIONAL** | Blocking checks OK, but warnings on deps/audit |
| **NO-GO** | At least one blocking check failed |

## Reports

### Structure

```
docs/audit/
  master.md                          # Central orchestration document
  audits/
    01-hexagonal-architecture.md     # Checklist template
    02-ddd.md
    03-clean-code.md
    ...
    12-performance.md
  reports/
    release-X.Y.Z/
      01-hexagonal-architecture.md   # Filled report
      02-ddd.md
      ...
      12-performance.md
      summary.md                     # Summary + final verdict
```

### Report contents

Each individual report contains:
- Compliance score table
- Findings by severity (CRIT/MAJ/MIN/INFO)
- Positive points
- Partial verdict (GO/CONDITIONAL/NO-GO)

The **summary** consolidates:
- All 12 scores
- Weighted global score
- Merged remediation PRs
- Remaining MAJ issues (prioritized for next cycle)
- The **final verdict**

## Real-world example

Release 0.3.1:
- 12 audits executed
- Initial scores variable (56-100 depending on axis)
- Final score: 95/100 (weighted)
- 5 remediation PRs merged
- 2 MAJOR issues remaining (scheduled for next cycle)
- **Verdict: GO**

## Red flags

- Audit skipped "because we're behind schedule" — the audit IS the schedule gate
- CRIT finding downgraded to MAJ without justification — severity must be evidence-based
- Audit report copy-pasted from previous release — each release gets a fresh audit
- Score artificially inflated by removing low-scoring items — the 12 axes are fixed
- No re-audit after remediation PRs — fixes must be verified

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "We audited last release, nothing changed much" | Code changed. Dependencies changed. The audit catches drift |
| "The score is 78, close enough to 80" | 78 is GO CONDITIONAL, not GO. Document the remediation plan |
| "This CRIT is theoretical, not exploitable" | If it's truly not exploitable, document why with evidence. Otherwise it's a CRIT |
| "Running 12 audits takes too long" | Automated checks run in < 2 min. Manual review is where quality lives |
| "The release gate CI covers everything" | CI checks syntax and known tests. Audit checks design, architecture, and intent |

## Verification

- [ ] All 12 axes audited (no skipped axes)
- [ ] Individual reports written for each axis
- [ ] Summary report with weighted global score
- [ ] All CRIT findings resolved (absolute NO-GO rule)
- [ ] MAJ findings either resolved or documented with remediation plan
- [ ] Verdict explicitly stated: GO / GO CONDITIONAL / NO-GO

## See also

- [Architecture checklist](../references/architecture-checklist.md) — Detailed hexagonal checks
- [Security checklist](../references/security-checklist.md) — OWASP + dependency audit
- [Performance checklist](../references/performance-checklist.md) — Profiling + thresholds
- [Testing patterns](../references/testing-patterns.md) — Coverage expectations + anti-patterns
