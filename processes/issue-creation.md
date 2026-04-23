# Issue Creation

> **When to use**: Every new issue opened on `scub-france/Docling-Studio`. Pair with [Issue Triage](issue-triage.md), which runs *after* an issue exists.

## The command

Use the Claude Code slash command `/create-issue` (defined in
[`.claude/commands/create-issue.md`](../.claude/commands/create-issue.md)) —
never hand-write `gh issue create` calls or use the GitHub web UI for new
issues. The command enforces the template, the type label, and the milestone.

Shape:

```
/create-issue <type> <milestone> <title>
```

The command **refuses** to create the issue if either constraint is missing.

## Mandatory constraints

| Constraint | Allowed values | Rationale |
|------------|----------------|-----------|
| **Type** (label) | `bug`, `feature`, `enhancement`, `documentation`, `test`, `chore` | Drives triage, changelog categorization, and audit axis mapping |
| **Milestone** | Any **open** milestone on the repo (`gh api repos/scub-france/Docling-Studio/milestones`) | Forces every ticket into a release window — no orphan backlog |
| **Title format** | `[TAG] Descriptive title` (see table below) | Makes issue lists scannable, enables prefix-based filters, matches the existing repo backlog |

Commit types (`feat`, `fix`, `docs`) are **not** issue types. The command maps
them (`feat→feature`, `fix→bug`, `docs→documentation`) or rejects them.

## Title convention

Every issue title **must** start with an uppercase tag in square brackets,
followed by a space and a descriptive noun phrase starting with a capital
letter. No trailing period.

| Type | Title tag | Example |
|------|-----------|---------|
| `bug` | `[BUG]` | `[BUG] CORS: methode PATCH absente de allow_methods` |
| `feature` | `[FEATURE]` | `[FEATURE] Panel to configure pipeline ingestion target` |
| `enhancement` | `[ENHANCEMENT]` | `[ENHANCEMENT] Reduire la taille de StudioPage.vue` |
| `documentation` | `[DOC]` | `[DOC] Analyser et mettre a jour la documentation projet` |
| `test` | `[TEST]` | `[TEST] Ajouter des tests composant pour ChunkPanel.vue` |
| `chore` | `[CHORE]` | `[CHORE] Bump ruff to 0.x` |

Additional tags seen in the backlog — **do not introduce new ones**, use the
mapping above:

- `[INFRA]` → covered by `chore` or the `infra` scope label
- `[FIX]` → alias of `[BUG]`, prefer `[BUG]`

The `/create-issue` command prepends the correct tag automatically when the
user-provided title lacks one. Do not hand-write `gh issue create` — the
convention drifts the moment someone bypasses the command.

### Label vs. tag mismatch

Some types have no matching label on the repo. In particular, there is no
`feature` label — the repo uses `enhancement` for both new features and
iterative improvements. In that case:

- The **title tag** still reflects the type (`[FEATURE]` for a new capability,
  `[ENHANCEMENT]` for an improvement to something existing).
- The **label** is the repo-valid one (`enhancement`).

Run `gh label list --repo scub-france/Docling-Studio` before creating the
issue if you are unsure.

## The template

The body always follows [`.github/ISSUE_TEMPLATE/issue.md`](https://github.com/scub-france/Docling-Studio/blob/main/.github/ISSUE_TEMPLATE/issue.md)
in the Docling Studio repo. Sections:

1. **Context** — why the issue exists
2. **Scope** — which subsystem (backend, frontend, infra, architecture, search, test, docs)
3. **Current behavior** — what happens today (or what's missing)
4. **Expected behavior** — what should happen
5. **Steps to reproduce / Acceptance criteria** — numbered or bulleted, verifiable
6. **Impact / Priority rationale** — ties to a `P0`/`P1`/`P2` label and, if relevant, an [audit axis](audit.md)
7. **References** — PRs, ADRs, logs, screenshots, related issues

Sections are never deleted. If a section isn't applicable, leave a one-line
`TODO:` so the assignee knows what to fill in.

## Workflow

1. **Decide the type first.** If you can't pick one of the six, the ticket
   probably bundles two issues — split them.
2. **Pick an open milestone.** If no existing milestone fits, create one
   *before* running `/create-issue`. Milestones are not auto-created.
3. **Run `/create-issue <type> <milestone> <title>`.** Provide a body inline
   or let the command open the template with `TODO:` placeholders.
4. **Verify the returned URL.** Confirm the type label, milestone, and
   template structure on GitHub before closing the loop.
5. **Hand off to [Issue Triage](issue-triage.md)** for priority (`P0`/`P1`/`P2`)
   and scope labels (`backend`, `frontend`, `infra`, etc.) within 48 hours.

## Red flags

- Issue opened without a type label — the command was bypassed
- Issue opened without a milestone — backlog rot starts here
- **Title missing the `[TAG]` prefix** or using a non-canonical tag (`[feat]`,
  `[improvement]`, `[Nice-to-have]`, etc.) — the command was bypassed or the
  user edited the title by hand afterwards
- **Title prefix in lowercase** (`[bug]` instead of `[BUG]`) — not our convention
- Body is a one-liner with no template sections — triage will stall
- Type set to `bug` but no reproduction steps — will bounce back with `needs-info`
- Commit-style types (`feat`, `fix`, `docs`) as labels — wrong vocabulary
- Same ticket covers two unrelated problems — should be two issues

## Anti-rationalizations

| Excuse | Why it doesn't hold |
|--------|---------------------|
| "I'll set the milestone later" | You won't. Untagged issues pile up in an untriaged bucket nobody revisits |
| "The type is obvious from the title" | Obvious to you today. Not to the triage bot, not to changelog generation, not to future-you |
| "The template is overkill for this one" | Templates are for the reader, not the writer. Three minutes now saves thirty in triage |
| "I'll just use the GitHub web UI, it's faster" | The web UI doesn't enforce type + milestone. That's the whole point of the command |
| "We don't have a milestone for this yet" | Then create one first. An issue without a milestone is a wish, not a plan |
| "The `[TAG]` prefix is just decoration" | It's how the backlog stays scannable. Search, filters, and changelog tooling key off the prefix — skip it once, and every subsequent list view has a hole |
| "My title is a full sentence, the prefix is redundant" | The prefix is for the reader skimming 200 issues, not for the author of this one. Keep the prefix, shorten the sentence |

## Verification

- [ ] Every open issue has exactly one type label (`bug`/`enhancement`/`documentation`/`test`/`chore` — note: no `feature` label on the repo, `[FEATURE]` titles carry `enhancement`)
- [ ] Every open issue has a milestone
- [ ] Every open issue title starts with a canonical `[TAG]` prefix (`[BUG]`, `[FEATURE]`, `[ENHANCEMENT]`, `[DOC]`, `[TEST]`, `[CHORE]`)
- [ ] Issue body uses all seven template sections (none deleted)
- [ ] `gh issue list --repo scub-france/Docling-Studio --search "no:milestone"` returns zero
- [ ] `gh issue list --repo scub-france/Docling-Studio --search "no:label"` returns zero
- [ ] `gh issue list --repo scub-france/Docling-Studio --state open --json title --jq '.[] | select(.title | test("^\\[(BUG|FEATURE|ENHANCEMENT|DOC|TEST|CHORE|INFRA|FIX)\\] ") | not)'` returns empty (no untagged titles)
