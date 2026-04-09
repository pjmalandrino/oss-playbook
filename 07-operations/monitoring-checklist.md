# Monitoring Checklist

What to monitor, what to alert on, and what to just log.

## The four golden signals

| Signal | What to measure | Alert when |
|--------|----------------|------------|
| **Latency** | Response time (P50, P95, P99) | P95 > <!-- REPLACE: e.g. 500ms --> |
| **Traffic** | Requests per second | Unusual spike or drop > <!-- REPLACE: e.g. 50% --> |
| **Errors** | Error rate (5xx, exceptions) | Error rate > <!-- REPLACE: e.g. 1% --> |
| **Saturation** | CPU, memory, disk, connections | Usage > <!-- REPLACE: e.g. 80% --> |

## Application-level checks

### Health endpoint

- [ ] `/health` or `/healthz` endpoint exists
- [ ] Checks database connectivity
- [ ] Checks external service connectivity
- [ ] Returns structured response:

```json
{
  "status": "healthy",
  "version": "2.1.0",
  "checks": {
    "database": "ok",
    "cache": "ok"
  }
}
```

### Key metrics to track

- [ ] Request count by endpoint and status code
- [ ] Response time by endpoint (histogram)
- [ ] Active users / sessions
- [ ] Background job queue depth and failure rate
- [ ] Database query time (slow query log)
- [ ] Cache hit/miss ratio

## Infrastructure checks

- [ ] Container health (restarts, OOM kills)
- [ ] Disk usage (especially for logs and uploads)
- [ ] SSL certificate expiry (alert 30 days before)
- [ ] DNS resolution
- [ ] Backup success/failure

## Alerting rules

### Alert (wake someone up)

- Service unreachable for > 2 minutes
- Error rate > <!-- REPLACE -->% for > 5 minutes
- Database connection failures
- Security scan findings (critical/high)
- SSL certificate expiring in < 14 days

### Warn (check next business day)

- Response time degradation > 2x baseline
- Disk usage > 80%
- Dependency deprecation warnings
- Failed background jobs

### Info (log, review weekly)

- Traffic pattern changes
- New error types appearing
- Dependency update available

## Tools

<!-- REPLACE: List your monitoring stack -->

| Purpose | Tool |
|---------|------|
| Metrics | <!-- REPLACE: e.g. Prometheus, Datadog --> |
| Logging | <!-- REPLACE: e.g. Grafana Loki, ELK --> |
| Alerting | <!-- REPLACE: e.g. PagerDuty, Grafana Alerting --> |
| Uptime | <!-- REPLACE: e.g. UptimeRobot, Pingdom --> |
| Error tracking | <!-- REPLACE: e.g. Sentry, Bugsnag --> |

## Dashboard

<!-- REPLACE: link to your primary monitoring dashboard -->
