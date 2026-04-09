# Deployment Checklist

## Pre-deploy

- [ ] Release branch merged to `main`
- [ ] Git tag created and pushed
- [ ] Build artifacts verified (correct version, no debug flags)
- [ ] Database migrations tested on staging
- [ ] Environment variables updated for production <!-- REPLACE: where are they managed? -->
- [ ] Feature flags configured <!-- REPLACE: if applicable -->
- [ ] Rollback plan reviewed (see [rollback-playbook.md](rollback-playbook.md))

## Deploy

- [ ] Deploy to production: <!-- REPLACE: describe deployment command/process -->
- [ ] Verify deployment succeeded (check logs, dashboard)
- [ ] Health check endpoints responding

## Post-deploy (smoke test)

- [ ] Core user flow works end-to-end <!-- REPLACE: describe the critical path -->
- [ ] API responds correctly (spot-check key endpoints)
- [ ] No error spike in monitoring <!-- REPLACE: link to dashboard -->
- [ ] No performance degradation
- [ ] Scheduled jobs still running <!-- REPLACE: if applicable -->

## Communication

- [ ] Status page updated (if applicable)
- [ ] Team notified <!-- REPLACE: channel -->
- [ ] Users notified (if user-facing change) <!-- REPLACE: channel -->

## Rollback trigger

Initiate rollback if any of these occur within the first **<!-- REPLACE: e.g. 30 -->** minutes:

- Error rate exceeds <!-- REPLACE: e.g. 1% --> of requests
- Response time exceeds <!-- REPLACE: e.g. 2x --> normal P95
- Any data corruption detected
- Critical user flow is broken
