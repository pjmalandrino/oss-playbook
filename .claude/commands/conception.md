---
description: Kick off a technical design (conception) phase on an existing issue — scaffold docs/design/<n>-<slug>.md from the repo template
argument-hint: <issue-number> [--force]
allowed-tools: Bash(gh issue view:*), Bash(gh issue list:*), Bash(gh issue comment:*), Bash(gh api:*), Bash(gh auth status:*), Bash(ls:*), Bash(test:*), Read, Write, Edit
---

# /conception — Start a technical design on an issue

Scaffold a **design doc** under `docs/design/` from `docs/design/TEMPLATE.md`,
pre-filled with data pulled from the target GitHub issue. This is the bridge
between a triaged issue and implementation — it produces the document that
drives the PR, and may spawn one or more ADRs.

## Hard constraints (non-negotiable)

1. **Issue number is required.** `/conception` with no argument must ask for
   one, not invent one.
2. **The issue must exist AND be open.** Closed/locked issues are refused —
   re-open or create a successor issue first.
3. **The issue must carry both a type label and a milestone.** If either is
   missing, refuse and point the user at `/create-issue` / the triage
   process. Designing against an untyped, milestone-less ticket is
   premature.
4. **The template file must exist** at `docs/design/TEMPLATE.md`. Do not
   hand-roll a body if it's missing — stop and report.
5. **Do not overwrite** an existing design doc for the same issue unless
   `--force` is passed.

## Arguments

User invocation: `/conception $ARGUMENTS`

Expected shape:

```
/conception <issue-number> [--force]
```

Examples:

- `/conception 142`
- `/conception 87 --force`

If no argument is provided, list the 10 most recent open issues with type +
milestone metadata and ask the user to pick one:

```bash
gh issue list --repo scub-france/Docling-Studio --state open \
  --limit 10 --json number,title,labels,milestone \
  --template '{{range .}}#{{.number}} [{{.milestone.title}}] {{.title}}{{"\n"}}{{end}}'
```

## Workflow

1. **Check `gh` auth.** Run `gh auth status`; if not authenticated, stop
   and tell the user to run `gh auth login`.

2. **Fetch the issue.**
   ```bash
   gh issue view <n> --repo scub-france/Docling-Studio \
     --json number,title,body,state,labels,milestone,url
   ```
   Refuse if `state != "OPEN"`.

3. **Validate metadata.**
   - Extract the type from labels. Exactly **one** of the repo-valid type
     labels must be present: `bug`, `enhancement`, `documentation`, `test`,
     `chore`. (There is no `feature` label — `[FEATURE]` titles carry the
     `enhancement` label; see `processes/issue-creation.md`.) Refuse if
     zero or more-than-one match.
   - Refuse if `milestone` is null.
   - **Validate the title tag.** The issue title must start with a
     canonical `[TAG]` prefix (`[BUG]`, `[FEATURE]`, `[ENHANCEMENT]`,
     `[DOC]`, `[TEST]`, `[CHORE]`). Warn (don't refuse) on `[INFRA]` /
     `[FIX]` — legacy but tolerated. Refuse on lowercase / missing /
     unknown prefixes and point at `/create-issue`.
   - Surface the detected type, title tag, milestone, and scope labels
     back to the user before writing anything.

4. **Compute the filename.** `docs/design/<n>-<slug>.md`, where `<slug>` is
   built from the issue title **after stripping the leading `[TAG]`
   prefix**: take the first 4-6 meaningful words, kebab-case, ASCII-only,
   drop stop-words. Example: issue #142 "[FEATURE] Expose batch progress
   via SSE" → `docs/design/142-expose-batch-progress-sse.md`.

5. **Refuse to overwrite.** If the target file already exists and `--force`
   was not passed, stop and show the existing path. With `--force`, replace
   it but warn clearly.

6. **Load `docs/design/TEMPLATE.md`.** Do not proceed if missing.

7. **Pre-fill the scaffold.** Replace the template's header placeholders
   with the values you just extracted:
   - `<short title>` → issue title **with the `[TAG]` prefix stripped**
     (the tag belongs on the issue, not on the design doc header)
   - `#<number>` → `#<n>`
   - `<copy the issue title>` → full issue title **including the `[TAG]`
     prefix** (keeps the cross-reference clean)
   - `**Author**` → current `git config user.name` (or blank if unavailable)
   - `**Date**` → today (ISO)
   - `**Target milestone**` → issue milestone
   - Section **1. Problem**: insert a short summary derived from the issue's
     Context + Current behavior blocks. Keep the user's voice; do not
     paraphrase aggressively.
   - Section **2. Goals**: convert the issue's acceptance criteria bullets
     into checkboxes. If the issue has none, leave the template's empty
     checkboxes untouched.
   - Section **12. References**: replace `Issue: <link>` with the real URL
     from `gh issue view`.

   All other sections are left as-is with their guidance comments. Do not
   invent content for §4-§11 — the point is for the author to fill them in.

8. **Write the file.** Use the `Write` tool. Never call `git` to stage or
   commit it automatically — that's the author's decision.

9. **Post a pointer comment on the issue** (optional, ask first):
   ```bash
   gh issue comment <n> --repo scub-france/Docling-Studio \
     --body "Design doc drafted at \`docs/design/<n>-<slug>.md\` — status: Draft."
   ```
   Skip this step if the user declines or if `--force` was used (assume
   re-scaffolding a stale doc doesn't deserve a new comment).

10. **Report.** Print the absolute path, the issue URL, the detected type +
    milestone, and a one-line next-step nudge ("fill in §3 Non-goals first
    — it's where designs die").

## Refusals and recovery

| Situation | Behavior |
|---|---|
| No arg passed | List recent open issues and ask |
| Issue not found / closed | Stop. Suggest re-opening or creating a new issue via `/create-issue` |
| No type label | Stop. Point at triage: "label the issue first with `bug` / `enhancement` / `documentation` / `test` / `chore`" |
| Title missing / bad `[TAG]` prefix | Stop. Point at `/create-issue` and the title convention in `processes/issue-creation.md` |
| No milestone | Stop. "Set a milestone before designing — see `processes/issue-creation.md`" |
| Multiple type labels | Stop. Ask the user to pick one |
| `docs/design/TEMPLATE.md` missing | Stop. Do not generate an ad-hoc template |
| Target path exists, no `--force` | Stop. Show the existing file so the user can decide |
| `gh` not authenticated | Stop. Tell user to `gh auth login` |

## Out of scope for this command

- Writing the actual design content (that's the author's job).
- Deciding whether the design warrants an ADR (see `processes/adr.md`).
- Opening a PR for the draft.
- Linking the design doc back to the issue body (only a comment is posted,
  and only with consent).

## Why this command exists

See `processes/conception.md` in the playbook for the full rationale,
red flags, and verification checklist.
