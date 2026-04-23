# Architecture Decision Records (ADR)

> **When to use**: Technology or framework choice, architectural pattern change, any decision with cross-cutting impact, or significant trade-off between options.

## Template

```markdown
# ADR-XXX: Decision title

**Date**: YYYY-MM-DD
**Status**: Proposed | Accepted | Deprecated | Superseded by ADR-YYY
**Deciders**: Name(s)

## Context
What problem or need triggered this decision? Include links to the related
design doc or issue, and list explicit constraints and non-goals.

## Decision
What solution was chosen?

## Consequences
### Positive
- ...

### Negative
- ...

### Neutral
- ...

## Alternatives Considered

### Alternative 1: Name
- **Pros**: ...
- **Cons**: ...
- **Why rejected**: ...

### Alternative 2: Name
- **Pros**: ...
- **Cons**: ...
- **Why rejected**: ...

## References
- Related design doc
- Related issue / PR
- External docs for the chosen option
```

## Naming convention

- File: `docs/architecture/adrs/ADR-XXX-short-title.md` (uppercase `ADR-` prefix, under `adrs/`)
- Sequential numbering (`ADR-001`, `ADR-002`, …)
- Short title in kebab-case

## Example in this repo

- [ADR-001: Graph visualization library for the Neo4j graph view](https://github.com/scub-france/Docling-Studio/blob/main/docs/architecture/adrs/ADR-001-graph-visualization-library.md) — chose Cytoscape.js after comparing vis-network, Neovis.js, D3, sigma.js/reagraph, and Mermaid.

## Other decisions that should become ADRs (retroactive)

- Choice of hexagonal architecture (ports & adapters)
- Choice of aiosqlite vs PostgreSQL
- Choice of Karate for E2E testing
- Dual Docker target (remote/local)
- Choice of Neo4j for graph storage (design doc: `docs/design/neo4j-integration.md`)

## Red flags

- Architecture decision made in a PR comment instead of an ADR — decisions get lost
- ADR written after implementation is done — it should guide the work, not document it after the fact
- No "Alternatives considered" section — proves the decision was a default, not a choice
- ADR with status "Proposed" for months — decide or discard
- Contradictory ADRs without superseding — mark old ones as deprecated

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "It's obvious, no need to write it down" | Obvious to you today, mysterious to the next person in 6 months |
| "An ADR is overkill for this" | If it affects more than one module, it deserves 15 minutes of writing |
| "We can always change it later" | True, but future-you needs to know WHY this choice was made to change it wisely |
| "The code is self-documenting" | Code shows what. ADRs show why, and what was rejected |

## Verification

- [ ] ADR written BEFORE or DURING implementation (not after)
- [ ] Context section explains the actual problem or need
- [ ] At least 2 alternatives considered with reasons for rejection
- [ ] Consequences include both positives AND negatives
- [ ] Status is current (not stuck at "Proposed")
- [ ] File follows naming convention (`docs/architecture/adrs/ADR-XXX-short-title.md`)
