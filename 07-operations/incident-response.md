# Incident Response Playbook

## Severity levels

| Level | Description | Examples | Target resolution |
|-------|-------------|----------|-------------------|
| **SEV-1** | Service down or data loss | App unreachable, database corruption | < 4 hours |
| **SEV-2** | Major feature broken | Auth failing, uploads broken, API errors > 5% | < 24 hours |
| **SEV-3** | Minor degradation | Slow responses, non-critical feature broken | < 72 hours |

## When an incident occurs

### 1. Assess

- What is the impact? (users affected, data at risk)
- What is the severity?
- When did it start? (check logs, monitoring)

### 2. Communicate

- Update status page (if applicable): <!-- REPLACE: link -->
- Notify stakeholders: <!-- REPLACE: channel -->
- For SEV-1/SEV-2: post a brief message immediately, don't wait for a diagnosis

### 3. Mitigate

Priority: **stop the bleeding first**, investigate later.

- Can you rollback? → See [rollback-playbook.md](../05-release/rollback-playbook.md)
- Can you disable the broken feature? (feature flag, config change)
- Can you scale resources to buy time?

### 4. Investigate

- Gather logs, metrics, error traces
- Identify the root cause (or most likely cause)
- Determine if this is a regression (check recent deployments)

### 5. Fix

- Implement the fix
- Test in staging if possible
- Deploy with extra monitoring

### 6. Resolve

- Confirm the fix works in production
- Update status page
- Notify stakeholders that the incident is resolved

---

## Post-mortem template

Write a post-mortem for every SEV-1 and SEV-2 incident.

### Incident: <!-- REPLACE: title -->

**Date**: <!-- REPLACE: YYYY-MM-DD -->
**Duration**: <!-- REPLACE: e.g. 2 hours 15 minutes -->
**Severity**: SEV-<!-- REPLACE -->
**Author**: <!-- REPLACE -->

#### Summary

<!-- REPLACE: 2-3 sentences. What happened and what was the impact? -->

#### Timeline

| Time (UTC) | Event |
|------------|-------|
| <!-- REPLACE --> | Issue first detected (how?) |
| <!-- REPLACE --> | Investigation started |
| <!-- REPLACE --> | Root cause identified |
| <!-- REPLACE --> | Fix deployed |
| <!-- REPLACE --> | Incident resolved |

#### Root cause

<!-- REPLACE: technical explanation of why this happened -->

#### What went well

- <!-- REPLACE -->

#### What went poorly

- <!-- REPLACE -->

#### Action items

| Action | Owner | Deadline | Status |
|--------|-------|----------|--------|
| <!-- REPLACE --> | <!-- REPLACE --> | <!-- REPLACE --> | Open |
| <!-- REPLACE --> | <!-- REPLACE --> | <!-- REPLACE --> | Open |

#### Lessons learned

<!-- REPLACE: What will we do differently to prevent this class of incident? -->
