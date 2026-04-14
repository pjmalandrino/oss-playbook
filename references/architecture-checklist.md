# Architecture Checklist (Hexagonal)

> Standalone reference checklist. Used during [code review](../processes/code-review.md) and [quality audits](../processes/audit.md).

## Domain layer (`domain/`)

- [ ] No imports from `api/`, `persistence/`, `infra/`, or `services/`
- [ ] No imports from FastAPI, SQLite, httpx, or any framework
- [ ] No I/O operations (no network, no file system, no database)
- [ ] Models are pure dataclasses or Pydantic BaseModel
- [ ] Value objects are immutable
- [ ] Business rules live here, not in services or API layer
- [ ] Ports (interfaces/protocols) are defined here

## API layer (`api/`) — Adapter IN

- [ ] Only FastAPI routers and Pydantic schemas
- [ ] Schemas use camelCase (frontend DTOs)
- [ ] No business logic — delegates to services
- [ ] Input validation via Pydantic (type, format, size)
- [ ] Error handling returns appropriate HTTP status codes
- [ ] No direct database or external service calls

## Services layer (`services/`)

- [ ] Orchestrates use cases by calling ports
- [ ] No framework-specific code (no FastAPI, no SQLAlchemy)
- [ ] No direct I/O — uses injected adapters
- [ ] Handles async job lifecycle (pending → running → done/failed)
- [ ] Business rule violations raise domain exceptions

## Persistence layer (`persistence/`) — Adapter OUT

- [ ] Implements repository interfaces defined in domain
- [ ] All database access is here (nowhere else)
- [ ] Uses parameterized queries (no string interpolation)
- [ ] Schema migrations are versioned
- [ ] No business logic — pure CRUD + queries

## Infrastructure layer (`infra/`) — Adapter OUT

- [ ] External service calls are here (converters, embeddings, etc.)
- [ ] Implements ports defined by domain
- [ ] Handles retries, timeouts, and error mapping
- [ ] Configuration loaded from environment variables (`settings.py`)
- [ ] No business logic — pure integration

## Dependency rules (the hexagonal contract)

```
ALLOWED                          FORBIDDEN
─────────                        ─────────
api/ → services/                 domain/ → api/
api/ → domain/ (models)          domain/ → persistence/
services/ → domain/              domain/ → infra/
services/ → persistence/ (port)  api/ → persistence/ (directly)
services/ → infra/ (port)        persistence/ → api/
persistence/ → domain/           infra/ → api/
infra/ → domain/
```

## Quick validation commands

```bash
# Check domain has no forbidden imports
grep -r "from api\|from persistence\|from infra\|from fastapi\|from aiosqlite" domain/

# Check no files exceed 300 lines
find . -name "*.py" -exec wc -l {} + | sort -rn | head -20

# Check no business logic in API layer
grep -r "if.*and\|for.*in\|while" api/ --include="*.py" | grep -v "schema\|import\|#"
```

## Common violations and fixes

| Violation | Example | Fix |
|-----------|---------|-----|
| Domain imports framework | `from fastapi import HTTPException` in `domain/` | Raise domain exception, let API layer map it |
| Business logic in router | `if document.pages > MAX_PAGES` in `api/` | Move validation to service or domain |
| Direct DB in service | `await db.execute(...)` in `services/` | Use repository port injection |
| API knows DB schema | SQL column names in `api/` | Map through DTOs in persistence layer |
| God service | Service with 500+ lines | Split by use case (upload, analyze, chunk) |
