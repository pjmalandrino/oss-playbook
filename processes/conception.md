# Conception (Technical Design)

> **When to use**: Any issue large enough that the implementation PR won't be obvious from the issue alone — new capability, cross-layer change, migration, performance rework, anything that could spawn an [ADR](adr.md). Runs **after** [Issue Creation](issue-creation.md) and [Triage](issue-triage.md), **before** implementation.

## The command

Use the Claude Code slash command `/conception` (defined in
[`.claude/commands/conception.md`](../.claude/commands/conception.md)) —
never hand-create design docs under `docs/design/`. The command enforces the
template, pulls metadata from the issue, and refuses to scaffold against an
untriaged ticket.

Shape:

```
/conception <issue-number> [--force]
```

With no argument, the command lists the 10 most recent open issues with
type + milestone metadata and asks the user to pick one.

## Mandatory preconditions

The command **refuses** to scaffold if any of these is missing:

| Precondition | Why |
|---|---|
| Issue exists and is **OPEN** | Designing against a closed issue hides the decision from the backlog |
| Issue has **exactly one** type label (`bug`/`enhancement`/`documentation`/`test`/`chore`) | Determines which audit axes and checklists apply |
| Issue has a **milestone** | Ties the design to a release window |
| Issue title carries a canonical `[TAG]` prefix | Matches the repo convention — see [Issue Creation](issue-creation.md) |
| `docs/design/TEMPLATE.md` exists in the checkout | Single source of truth for the doc structure |
| No existing design doc for the same issue (unless `--force`) | Prevents silent overwrites of work in progress |

## The template

The scaffold follows [`docs/design/TEMPLATE.md`](https://github.com/scub-france/Docling-Studio/blob/main/docs/design/TEMPLATE.md)
in the Docling Studio repo. Sections:

1. **Problem** — restated from the issue
2. **Goals** — verifiable, mirroring the acceptance criteria
3. **Non-goals** — the most load-bearing section (scope creep dies here)
4. **Constraints** — hexagonal boundaries, frontend feature layout, dual Docker target, perf/security budgets
5. **Current state** — what already exists in the codebase
6. **Proposed design** — overview, architecture impact, data model, API, frontend, configuration
7. **Alternatives considered** — at least one real alternative with real trade-offs
8. **Migration & rollout** — feature flag, phased plan, rollback path
9. **Testing strategy** — unit, `pytestarch`, Karate `@smoke`/`@regression`/`@critical`, manual verification
10. **Observability** — logs, metrics, traces, oncall signal
11. **Risks & open questions** — unknowns and what could go wrong
12. **References** — issue, PRs, prior designs, external docs

Sections are never deleted. Empty ones keep their guidance comments so the
next reader knows what belongs there.

## Filename convention

`docs/design/<issue-number>-<kebab-slug>.md`

- `<issue-number>` pins the design to exactly one issue.
- `<kebab-slug>` is derived from the issue title **after stripping the
  `[TAG]` prefix**, ASCII-only, 4–6 meaningful words, no stop-words.

Example: issue #142 "[FEATURE] Expose batch progress via SSE"
→ `docs/design/142-expose-batch-progress-sse.md`.

## Workflow

1. **Make sure the issue is ready.** Type label + milestone + `[TAG]` title.
   If any is missing, fix it with `/create-issue` or triage first — do not
   start the design in parallel.
2. **Run `/conception <n>`.** The command validates, scaffolds, and
   (optionally) posts a pointer comment on the issue.
3. **Fill in `§3 Non-goals` first.** Literally. It's the cheapest and
   highest-leverage section — if you can't name what's out of scope, the
   design isn't ready.
4. **Flip status to `Review`** when §1–§9 are filled, and share the doc.
   Review happens on the doc, not in issue comments.
5. **Spawn an ADR if needed.** If the design locks in a cross-cutting
   architectural choice (new port, new adapter, framework switch, storage
   swap), write the corresponding `ADR-XXX` in `docs/architecture/adrs/`.
   The ADR is short; the design doc is the long form.
6. **Flip to `Accepted`** before implementation starts. `Implemented` when
   the PR lands.

## Red flags

- Design doc created without a linked issue — orphan document, triage can't find it
- Implementation PR opened before the design is `Accepted` — reviews will drift into re-designing in the PR
- `§3 Non-goals` is empty or says "N/A" — scope will explode during implementation
- `§7 Alternatives considered` lists only one option — proves the decision was a default, not a choice
- `§11 Risks & open questions` is empty — either the change is trivial (no design doc needed) or unknowns are hidden
- Status stuck at `Draft` for weeks — close the design or close the issue, don't let both rot
- Design doc edited after the PR merges without flipping status to `Implemented` / `Superseded` — future readers can't tell what's current

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "I know what I'm going to build, a doc is overhead" | The doc is for the reviewer and for future-you. If you know the answer, writing it costs 20 minutes |
| "I'll write it after I prototype" | Design-after-code becomes post-hoc justification. The whole point is to front-load the trade-offs |
| "This doesn't need an ADR, so it doesn't need a design either" | ADR ≠ design. ADRs capture one decision; designs capture a whole change shape. Many designs have zero ADRs |
| "The issue already describes the change" | The issue describes the *problem*. The design describes the *solution* — and non-goals, alternatives, rollout. Different documents |
| "We'll figure out testing during implementation" | Testing strategy written *after* the code is written is always shaped by the code, not by the requirements |

## Verification

- [ ] Every `docs/design/*.md` (except `TEMPLATE.md`) starts with `<issue-number>-`
- [ ] Every design doc references an issue in §12 and has a reciprocal comment on the issue
- [ ] No design doc sits at status `Draft` for more than 2 weeks
- [ ] `§3 Non-goals` and `§7 Alternatives considered` are never empty on an `Accepted` design
- [ ] Every merged PR larger than ~300 LOC traces back to either a design doc or a justified exemption in the PR description
- [ ] `ls docs/design/ | grep -vE '^(TEMPLATE\.md|[0-9]+-.+\.md)$'` returns empty
