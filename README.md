# Docling Studio — Developer Playbook

> My complete development playbook for [Docling Studio](https://github.com/scub-france/Docling-Studio), shared for full transparency.
> This repo documents the processes, tools, workflows, and conventions I use daily.

## About this playbook

This repo contains **practices and processes only**. The following files live in the main Docling Studio repository:
- `CLAUDE.md` — Per-subproject conventions (in `document-parser/`, `frontend/`)
- `CONTRIBUTING.md` — Contribution guidelines
- `CHANGELOG.md` — Version history
- `docs/` — Full documentation (MkDocs, deployed on GitHub Pages)

---

## Where to start?

### Day 1 — Setup and first launch
1. [Onboarding: prerequisites and installation](processes/onboarding.md)
2. [Run the project](processes/onboarding.md#run-the-project)
3. [Understand the architecture](#backend-architecture-hexagonal--ports--adapters) (Hexagonal, feature-based)

### First week — Start developing
4. [Commit conventions](processes/commit-conventions.md) then [Code review checklist](processes/code-review.md)
5. [Testing strategy](quality/testing-strategy.md) — What to test and how
6. [Local development workflow](#1-local-development) — Lint, format, test before every commit

### When you need it
- Deploy a release → [Release](processes/release.md)
- Critical bug in production → [Incident](processes/incident-response.md) then [Hotfix](processes/hotfix.md) or [Rollback](processes/rollback.md)
- Architecture decision → [ADR](processes/adr.md)
- Open a new ticket → [Issue Creation](processes/issue-creation.md) (`/create-issue`)
- Kick off a technical design on an issue → [Conception](processes/conception.md) (`/conception`)
- Start implementing an issue → [Resolve](processes/resolve.md) (`/resolve`)
- Understand the CI → [CI Pipeline](ci-cd/ci.md)

---

## Overview

This playbook covers a full-stack project composed of:

| Layer | Stack | Quality tools |
|-------|-------|---------------|
| **Backend** | FastAPI + Python 3.12 + aiosqlite | Ruff (lint + format), pytest, pytestarch (hexagonal rules) |
| **Frontend** | Vue 3 + TypeScript (strict) + Vite + Pinia + Cytoscape.js | ESLint 9 (flat config) + Prettier, Vitest |
| **Embedding service** | FastAPI + sentence-transformers (Python, opt-in) | Ruff, pytest |
| **Ingestion / Search** | OpenSearch (vector + full-text, opt-in) | Docker Compose overlay |
| **Graph storage** | Neo4j (DoclingDocument tree + chunks, opt-in) | Bolt driver, Cypher |
| **Live reasoning** | `docling-agent` + Ollama (opt-in, `RAG_ENABLED=true`) | Chunkless RAG loop |
| **E2E** | Karate (API) + Karate UI (browser) | Maven, Chrome headless |
| **Infra** | Docker multi-target, Nginx, docker-compose | GitHub Actions CI/CD |
| **Docs** | MkDocs Material | GitHub Pages |

---

## Existing workflows

### 1. Local development

```
Code -> Lint/Format -> Unit tests -> Commit (Conventional Commits)
```

**Backend:**
```bash
ruff check . --fix && ruff format .    # Lint + format
ruff check . && ruff format --check .  # Verify
pytest tests/ -v                       # Tests (377+ tests)
```

**Frontend:**
```bash
npm run lint:fix && npm run format     # Lint + format
npm run lint && npm run format:check   # Verify
npm run type-check                     # vue-tsc --noEmit
npm run test:run                       # Vitest (156+ tests)
```

### 2. CI pipeline (GitHub Actions)

```
PR opened
  |
  +-> Backend: ruff check + pytest
  +-> Frontend: eslint + vue-tsc + vitest + vite build
  |
  +-> E2E API (@smoke on PR, @smoke+@regression+@e2e on release PR)
  +-> E2E UI (@critical on main only)
```

### 3. Quality audit (pre-release)

```
release/* branch ready
  |
  +-> Phase 1 (parallel):
  |     Lint + Tests + Dep audit (pip-audit, npm audit) + Audit checks (12-axis script)
  |
  +-> Phase 2:
  |     Docker build (remote + local) + Smoke test + Trivy scan + Image size check
  |
  +-> Phase 3:
  |     E2E API (@smoke + @regression + @e2e) + E2E UI (@critical)
  |
  +-> Phase 4:
        Automatic verdict on PR: GO / GO CONDITIONAL / NO-GO
```

**12 audited axes**: Hexagonal Architecture, DDD, Clean Code, KISS, DRY, SOLID, Decoupling, Security, Tests, CI/Build, Documentation, Performance.

**Scoring**: each axis is weighted, score >= 80 = GO, any `[CRIT]` = absolute NO-GO.

See [full audit process details](processes/audit.md).

### 4. Release

```
main -> release/X.Y.Z -> Audit GO -> merge main -> tag vX.Y.Z
  |
  +-> Docker build multi-arch (amd64 + arm64)
  +-> 2 targets: remote (lightweight) / local (with AI models)
  +-> Push to ghcr.io with semantic tags (X.Y.Z, X.Y, latest)
```

### 5. Hotfix

```
tag vX.Y.Z -> hotfix/X.Y.Z+1 -> PR to main -> tag vX.Y.Z+1
```

---

## Which process should I use?

```
You have a change to make
  |
  +-> New feature or enhancement?
  |     -> feature/* branch -> PR to main -> [Commit] [Code Review] [Merge]
  |
  +-> Non-critical bug fix?
  |     -> fix/* branch -> PR to main -> [Commit] [Code Review] [Merge]
  |
  +-> Critical production bug?
  |     +-> Fix is obvious and quick?
  |     |     -> [Hotfix]: hotfix/* from tag -> PR to main -> tag
  |     +-> Fix is complex or uncertain?
  |           -> [Rollback] first, then fix via regular PR
  |
  +-> Release time?
  |     -> [Release]: release/* branch -> [Audit] -> merge main -> tag
  |
  +-> Architecture decision?
        -> [ADR]: document in docs/architecture/
```

## Available processes

> Every process follows a consistent structure: **When to use → Steps → Red flags → Anti-rationalizations → Verification**. Red flags tell you when the process is being violated. Anti-rationalizations counter the common excuses for skipping steps. Verification proves the process was followed.

| # | Process | Trigger | Details |
|---|---------|---------|---------|
| 1 | [Commit](processes/commit-conventions.md) | Every commit | Strict Conventional Commits |
| 2 | [Code Review](processes/code-review.md) | Every PR | Structured checklist |
| 3 | [Merge Policy](processes/merge-policy.md) | PR ready | Merge rules |
| 4 | [ADR](processes/adr.md) | Architecture decision | Architecture Decision Records |
| 5 | [**Quality Audit**](processes/audit.md) | Before merge release -> main | 12 axes, weighted scoring, CRIT/MAJ/MIN/INFO |
| 6 | [Release](processes/release.md) | Feature freeze | Branch, tag, deploy |
| 7 | [Hotfix](processes/hotfix.md) | Critical bug in production | Quick patch |
| 8 | [Rollback](processes/rollback.md) | Post-deploy incident | Revert to previous version |
| 9 | [Incident Response](processes/incident-response.md) | Incident detected | Triage, resolution, post-mortem |
| 10 | [Security Response](processes/security-response.md) | Vulnerability detected | Assessment, patch, disclosure |
| 11 | [Onboarding](processes/onboarding.md) | New contributor | Getting started guide |
| 12 | [Issue Triage](processes/issue-triage.md) | New issue | Classification and prioritization |
| 13 | [Issue Creation](processes/issue-creation.md) | Opening a new ticket | `/create-issue` slash command — enforces required type + milestone |
| 14 | [Conception](processes/conception.md) | Before implementing a non-trivial issue | `/conception` slash command — scaffolds `docs/design/<n>-<slug>.md` from the template |
| 15 | [Resolve](processes/resolve.md) | Start implementing an issue | `/resolve` slash command — validates metadata + design, creates `<prefix>/<n>-<slug>` branch off `main` |

---

## Detailed tech stack

### Backend architecture (Hexagonal — Ports & Adapters)

```
          ┌─────────────────────────────┐
          │        domain/              │  <- The core (hexagon)
          │  Pure business logic        │     No external dependencies
          │  Models, value objects      │     Ports = interfaces
          │  Ports: DocumentConverter,  │
          │  VectorStore, EmbeddingSvc, │
          │  GraphWriter, …             │
          └──────────┬──────────────────┘
                     │
       ┌─────────────┼─────────────────────┐
       ▼             ▼                     ▼
   api/          services/          persistence/
   Adapter IN    Orchestration      Adapter OUT
   FastAPI       Use cases          Repository pattern
   Pydantic DTOs                    (aiosqlite)
   (camelCase)
                                    infra/
                                    Adapters OUT
                                    Local/Serve converters,
                                    Neo4j writers,
                                    OpenSearch store,
                                    Embedding HTTP client,
                                    rate limiter, settings
```

**Principle**: the domain depends on nothing. Adapters (API, persistence, infra) implement ports defined by the domain. Any adapter can be swapped without touching the core.

**Enforcement**: hexagonal rules are encoded as [`pytestarch`](https://github.com/zyskarch/pytestarch) tests in `document-parser/tests/test_architecture.py` — CI fails if any import crosses a forbidden boundary (domain → framework, services → infra concrete, etc.).

### Frontend architecture (Feature-based)

```
src/
  app/         -> Application shell, router, global styles
  pages/       -> Route pages
  features/    -> Feature modules
    {name}/
      api/     -> HTTP calls
      store/   -> Pinia state
      ui/      -> Vue components
  shared/      -> Utilities, types, i18n, HTTP client
```

### Docker infrastructure

| Target | Contents | Use case |
|--------|----------|----------|
| `remote` | Lightweight backend, delegates parsing via HTTP | When an external conversion service exists |
| `local` | Backend + embedded AI models (PyTorch CPU) | Standalone, all-in-one |

**Compose stacks:**

| Stack | File | Services |
|-------|------|----------|
| Production | `docker-compose.yml` | app + nginx |
| Development | `docker-compose.dev.yml` | + hot-reload (uvicorn --reload, vite HMR), OpenSearch Dashboards |
| Ingestion (opt-in) | `docker-compose.ingestion.yml` | + OpenSearch + embedding service + **Neo4j** |

See [infrastructure/docker.md](infrastructure/docker.md) for the full env-var table (Neo4j, RAG, rate limiting, upload caps).

---

## Quality configuration

See details in:

- [Ruff (backend)](quality/ruff-config.md)
- [ESLint + Prettier (frontend)](quality/eslint-prettier-config.md)
- [Testing Strategy](quality/testing-strategy.md)
- [E2E Conventions](quality/e2e-conventions.md)

---

## Reference checklists

Standalone, actionable checklists used during code reviews and audits:

| Checklist | Used during | What it covers |
|-----------|-------------|----------------|
| [Architecture (Hexagonal)](references/architecture-checklist.md) | Code review, audit axis 01 | Dependency rules, layer violations, quick validation commands |
| [Security](references/security-checklist.md) | Code review, audit axis 08 | OWASP Top 10, input validation, secrets, dependencies |
| [Performance](references/performance-checklist.md) | Code review, audit axis 12 | Async I/O, N+1, bundle size, measurement thresholds |
| [Testing Patterns](references/testing-patterns.md) | Code review, audit axis 09 | Naming, structure, anti-patterns, coverage expectations |

---

## Claude Code integration

I use Claude Code as a development assistant. See:

- [Claude Code Setup](claude-code/setup.md) — Configuration, CLAUDE.md, permissions
- [Claude Code Workflows](claude-code/workflows.md) — Automated validation pipelines

---

## CI/CD configuration

See details in:

- [CI Pipeline](ci-cd/ci.md)
- [Release Pipeline](ci-cd/release.md)
- [Release Gate](ci-cd/release-gate.md)

---

## Versioning

- **Semantic Versioning**: MAJOR.MINOR.PATCH
- **Source of truth**: Git tag `vX.Y.Z`
- **Auto-injection**: Vite `__APP_VERSION__` (frontend), `APP_VERSION` env var (backend)
- **Changelog**: `CHANGELOG.md` following Keep a Changelog format
