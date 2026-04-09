# Security Vulnerability Response Process

How to handle security vulnerability reports.

## Receiving a report

Vulnerabilities should be reported via:

- Email: <!-- REPLACE: security contact -->
- GitHub Security Advisories: <!-- REPLACE: link if enabled -->

**Never** discuss unpatched vulnerabilities in public issues.

## Response timeline

| Step | Target time | Action |
|------|------------|--------|
| Acknowledge | < 24 hours | Reply to reporter confirming receipt |
| Assess | < 48 hours | Determine severity and affected versions |
| Fix | < 7 days | Develop and test the patch |
| Release | < 14 days | Publish patched version |
| Disclose | After release | Publish security advisory |

## Severity assessment

Use [CVSS](https://www.first.org/cvss/) or this simplified scale:

| Severity | Description | Examples |
|----------|-------------|----------|
| **Critical** | Remote code execution, auth bypass, data exfiltration | SQL injection, RCE, broken auth |
| **High** | Significant data exposure or privilege escalation | IDOR, XSS with session theft |
| **Medium** | Limited impact, requires specific conditions | CSRF, information disclosure |
| **Low** | Minimal impact | Verbose error messages, minor info leak |

## Process

### 1. Acknowledge

```
Thank you for reporting this. We take security seriously and will
investigate immediately. We'll update you within 48 hours with our
assessment.
```

### 2. Assess

- [ ] Reproduce the vulnerability
- [ ] Determine affected versions
- [ ] Assess severity (CVSS or simplified scale above)
- [ ] Identify the root cause

### 3. Fix

- [ ] Develop the patch in a **private branch** (or GitHub Security Advisory fork)
- [ ] Write tests that verify the fix
- [ ] Review the fix (even solo — use the code review checklist)
- [ ] **Do not push to a public branch until the advisory is ready**

### 4. Release

- [ ] Bump the patch version
- [ ] Update CHANGELOG.md (describe the fix without exposing exploit details)
- [ ] Publish the release
- [ ] Tag affected versions in the advisory

### 5. Disclose

- [ ] Publish a GitHub Security Advisory
- [ ] Credit the reporter (unless they prefer anonymity)
- [ ] Request a CVE if appropriate
- [ ] Announce in project communication channels

## Template: Security Advisory

```markdown
## Summary

A [severity] vulnerability was discovered in [component] that allows
[impact description].

## Affected versions

- [version range]

## Patched versions

- [version]

## Workarounds

[If any, describe. Otherwise: "Upgrade to the patched version."]

## Credit

Reported by [reporter] (if they agreed to be credited).
```
