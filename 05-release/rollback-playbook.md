# Rollback Playbook

What to do when a release goes wrong.

## Decision: rollback vs. hotfix

| Situation | Action |
|-----------|--------|
| Bug is critical AND fix is not obvious | **Rollback** |
| Bug is critical BUT fix is a one-liner | **Hotfix** |
| Bug is major but not urgent | **Hotfix** in next patch release |
| Data corruption detected | **Rollback immediately** |

## Rollback procedure

### 1. Announce

Notify the team that a rollback is in progress:

```
<!-- REPLACE: e.g. post in #deploys Slack channel -->
```

### 2. Revert the deployment

```bash
# <!-- REPLACE: your rollback command -->
# Examples:
# Docker: docker compose up -d --pull always  (with previous image tag)
# Kubernetes: kubectl rollout undo deployment/<name>
# Heroku: heroku rollback
# AWS ECS: update service with previous task definition
```

### 3. Verify

- [ ] Health checks pass
- [ ] Core user flow works
- [ ] Error rate back to baseline
- [ ] Monitoring dashboards normal

### 4. Database considerations

If the release included database migrations:

- [ ] Are the migrations backward-compatible? (Can old code work with new schema?)
- [ ] If not, apply a reverse migration: <!-- REPLACE: command -->
- [ ] Verify data integrity after rollback

### 5. Post-rollback

- [ ] Create an issue documenting what went wrong
- [ ] Identify the root cause
- [ ] Write the fix
- [ ] Add tests that would have caught the issue
- [ ] Re-release with the fix

## Hotfix procedure

1. Branch from `main`: `git checkout -b hotfix/<description>`
2. Fix the issue (minimal change)
3. Add a test for the regression
4. Fast-track code review
5. Merge to `main`, tag, deploy
6. Follow the normal deployment checklist

## Post-incident

After any rollback or hotfix, write a brief incident report:

- **What happened**: one paragraph
- **Timeline**: when detected, when resolved
- **Root cause**: why the bug shipped
- **What we'll do differently**: action items

See [07-operations/incident-response.md](../07-operations/incident-response.md) for the full template.
