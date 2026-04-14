# Incident Response

> **When to use**: Production issue detected — service degraded or down.

## Severity

| Level | Criteria | Response time |
|-------|----------|---------------|
| **P1 — Critical** | Service down, data loss | Immediate |
| **P2 — Major** | Core feature broken | < 4h |
| **P3 — Minor** | Secondary feature degraded | < 24h |
| **P4 — Cosmetic** | UI glitch, error message | Next sprint |

## Workflow

### 1. Detection and triage

- Identify severity
- Assign an owner
- Communicate status

### 2. Investigation

- Check application logs
- Review metrics (if monitoring is in place)
- Reproduce the issue locally if possible

### 3. Resolution

- **P1**: Immediate [rollback](rollback.md) if fix is not obvious
- **P2-P3**: [Hotfix](hotfix.md) following the standard process
- **P4**: Issue created, scheduled in backlog

### 4. Post-mortem

For any P1/P2 incident:

```markdown
## Post-mortem: [Title]

### Timeline
- HH:MM — Detection
- HH:MM — Investigation started
- HH:MM — Root cause identified
- HH:MM — Fix deployed
- HH:MM — Resolution confirmed

### Root cause
...

### Impact
- Duration: X hours
- Affected users: ...

### Corrective actions
- [ ] Technical fix
- [ ] Additional test
- [ ] Monitoring improvement
```

## Red flags

- P1 incident without immediate rollback attempt — rollback first, investigate later
- No post-mortem written after P1/P2 — same incident will happen again
- Post-mortem assigns blame to individuals — focus on systems, not people
- Incident resolved but no test added — the regression will return
- Same root cause appearing in multiple incidents — systemic issue being ignored

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "Let me debug it in production, I'll find the fix faster" | Debugging live while users are impacted is never faster. Rollback first |
| "It only affects a few users" | A few users today is all users tomorrow. Severity doesn't decrease on its own |
| "We don't have time for a post-mortem" | The 30 minutes you save now cost 4 hours when the same bug returns |
| "The fix is deployed, we're done" | Without corrective actions (test, monitoring), you're one deploy away from repeat |

## Verification

- [ ] Severity correctly assessed and communicated
- [ ] Owner assigned within response time SLA
- [ ] Resolution applied (rollback/hotfix/scheduled fix)
- [ ] Production health confirmed post-resolution
- [ ] Post-mortem written for P1/P2 (within 48 hours)
- [ ] Corrective actions created as issues with owners
