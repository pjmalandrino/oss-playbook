# 07 — Operations

Playbooks for when things go wrong in production.

## What's here

| Template | Purpose |
|----------|---------|
| [incident-response.md](incident-response.md) | Step-by-step incident handling + post-mortem template |
| [security-response.md](security-response.md) | CVE handling and coordinated disclosure process |
| [monitoring-checklist.md](monitoring-checklist.md) | What to monitor and alert on |

## When to use these

- **incident-response.md** — when users report something is broken in production
- **security-response.md** — when someone reports a security vulnerability
- **monitoring-checklist.md** — when setting up your project for production readiness

## Solo maintainer note

You might not have a 24/7 on-call rotation. That's fine. These playbooks help you respond *methodically* when you do have time, instead of panicking and making things worse.
