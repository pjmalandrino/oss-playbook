# ADR-001 — Use PostgreSQL as the primary database

**Date**: 2026-01-15
**Status**: Accepted

## Context

The application needs a persistent data store for documents, analyses, and user sessions. We currently use SQLite during development, but it won't scale to production with concurrent users.

We need a database that supports:

- ACID transactions
- JSON queries (for document metadata)
- Full-text search (for document content)
- Concurrent connections (50+ expected)

## Decision

We will use **PostgreSQL 16** as the primary database for all environments (dev, staging, production).

Development will use a Docker-based PostgreSQL instance to match production. SQLite will no longer be supported.

## Consequences

### Positive

- PostgreSQL's JSONB support eliminates the need for a separate document store
- Built-in full-text search avoids adding Elasticsearch for our current scale
- Mature ecosystem with excellent Python (asyncpg, SQLAlchemy) and Java (JDBC, Hibernate) drivers
- Easy to run in Docker for local development

### Negative

- Contributors must run Docker (or install PostgreSQL locally) to develop
- Slightly more complex CI setup than SQLite
- Operational burden: backups, connection pooling, upgrades

### Risks

- If we outgrow PostgreSQL's full-text search, we'll need to add Elasticsearch later (medium likelihood, low effort to migrate)

## Alternatives considered

### SQLite

Simple, zero-config, but no concurrent write support. Works for prototyping, not for production with multiple users.

### MongoDB

Good JSON support, but we lose ACID transactions across collections. Our data model is relational, and the team has more SQL experience.

### MySQL

Would work, but PostgreSQL's JSONB and full-text search give us more flexibility for our document-heavy workload.
