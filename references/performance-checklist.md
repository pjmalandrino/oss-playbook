# Performance Checklist

> Standalone reference checklist. Used during [code review](../processes/code-review.md) and [quality audits](../processes/audit.md).

## Backend

### Async I/O
- [ ] All database operations use `async`/`await` (aiosqlite)
- [ ] External HTTP calls use `httpx.AsyncClient` (not `requests`)
- [ ] File I/O uses async where available
- [ ] No blocking calls in async handlers (no `time.sleep()`)

### Query patterns
- [ ] No N+1 queries (fetching related data in loops)
- [ ] Bulk operations used where possible (batch insert/update)
- [ ] Database queries are indexed for common access patterns
- [ ] Large result sets are paginated

### Concurrency
- [ ] `MAX_CONCURRENT_ANALYSES` respected (no unbounded parallelism)
- [ ] Background tasks use proper async task management
- [ ] Rate limiting prevents resource exhaustion (`RATE_LIMIT_RPM`)

### Memory
- [ ] Large files streamed, not loaded entirely in memory
- [ ] Temporary files cleaned up after processing
- [ ] No unbounded caches or growing collections

## Frontend

### Bundle size
- [ ] No unnecessary dependencies (check `npm ls --all`)
- [ ] Dynamic imports (`lazy()`) for route-level code splitting
- [ ] Tree-shaking friendly imports (named imports, not barrel files)

### Rendering
- [ ] Lists use `v-for` with `:key` (no index-as-key on dynamic lists)
- [ ] Heavy computations use `computed` (not recalculated on every render)
- [ ] Large lists use virtual scrolling if > 100 items
- [ ] Images are lazy-loaded where appropriate

### Network
- [ ] API calls are debounced for user input (search, filters)
- [ ] No redundant API calls (check Pinia stores for deduplication)
- [ ] Loading states shown during async operations

## Docker

- [ ] Multi-stage build keeps image size minimal
- [ ] `remote` target doesn't include AI model dependencies
- [ ] Build cache used effectively (dependency layers before code layers)
- [ ] Health check endpoints respond in < 100ms

## Measurement

### How to profile

```bash
# Backend — request timing
time curl -s http://localhost:8000/api/health

# Backend — endpoint profiling
pytest tests/ -v --durations=10        # Slowest tests

# Frontend — bundle analysis
npm run build -- --report             # Vite bundle visualizer

# Docker — image size
docker images | grep app
```

### Acceptable thresholds

| Metric | Target | Alert |
|--------|--------|-------|
| API health check | < 50ms | > 200ms |
| Document upload (10 pages) | < 2s | > 5s |
| Frontend initial load | < 1.5s | > 3s |
| Frontend route navigation | < 300ms | > 1s |
| Docker image (remote) | < 500MB | > 800MB |
| Docker image (local) | < 4GB | > 6GB |
