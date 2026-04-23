---
description: Create a Docling Studio issue from the repo template with a required type and milestone
argument-hint: <type> <milestone> [title...]
allowed-tools: Bash(gh issue create:*), Bash(gh api:*), Bash(gh label list:*), Bash(gh repo view:*), Read, Write
---

# /create-issue — Open a Docling Studio issue

Create a GitHub issue on **scub-france/Docling-Studio** that strictly follows
[`.github/ISSUE_TEMPLATE/issue.md`](https://github.com/scub-france/Docling-Studio/blob/main/.github/ISSUE_TEMPLATE/issue.md) and carries two mandatory pieces of metadata: a **type** label
and a **milestone**. Anything else is rejected before calling `gh`.

## Hard constraints (non-negotiable)

1. **Type is required** — exactly one of:
   `bug` · `feature` · `enhancement` · `documentation` · `test` · `chore`
   Passed as a GitHub label (`--label <type>`).
2. **Milestone is required** — must match an **open** milestone on the repo.
   Passed as `--milestone <name>`.
3. **Title must follow the repo convention**: `[TAG] Descriptive title`
   where `TAG` is the uppercase form of the type:

   | Type | Title tag |
   |------|-----------|
   | `bug` | `[BUG]` |
   | `feature` | `[FEATURE]` |
   | `enhancement` | `[ENHANCEMENT]` |
   | `documentation` | `[DOC]` |
   | `test` | `[TEST]` |
   | `chore` | `[CHORE]` |

   The descriptive part starts with a capital letter and is written as a
   short noun phrase (no trailing period). If the user-provided title is
   missing the `[TAG]` prefix, **prepend it automatically** based on the
   resolved type — do not ask. If the user already provided a prefix,
   normalize casing and keep their wording.
4. **Label mapping when a type has no matching repo label** — the repo may
   not have a label for every playbook type (e.g. `feature` is absent; the
   repo uses `enhancement`). When the requested type has no matching label,
   pause and ask the user which existing label to apply. The **title tag**
   still follows the type (`[FEATURE]` stays `[FEATURE]`), only the
   `--label` value is substituted.
5. If any of the above is missing, empty, or unknown, **do not create the
   issue**. Ask the user for the missing value instead.

## Arguments

User invocation: `/create-issue $ARGUMENTS`

Expected shape:

```
/create-issue <type> <milestone> <title>
```

Examples (the command auto-prepends the `[TAG]` if absent):

- `/create-issue bug 0.7.0 Upload fails on files > 50MB`
  → title: `[BUG] Upload fails on files > 50MB`
- `/create-issue feature 1.0.0 Expose batch progress via SSE`
  → title: `[FEATURE] Expose batch progress via SSE`
- `/create-issue documentation 0.6.0 Document Neo4j env vars in infrastructure/docker.md`
  → title: `[DOC] Document Neo4j env vars in infrastructure/docker.md`

If the user runs `/create-issue` with no arguments, prompt for them one by one
in this order: type → milestone → title → body.

## Workflow

1. **Parse `$ARGUMENTS`.** Extract `type`, `milestone`, and the remaining
   words as the title. Trim quotes.

2. **Validate the type.** Must be exactly one of the six allowed values.
   Reject anything else (including `feat`, `fix`, `docs`, etc. — those are
   *commit* types, not *issue* types; map them: `feat→feature`, `fix→bug`,
   `docs→documentation`).

3. **Normalize the title.** If the user-provided title does not already
   start with a `[TAG]` prefix matching the resolved type, prepend it
   according to the mapping in the hard constraints. Capitalize the first
   letter of the descriptive part. Example: `panel to configure X` with
   type `feature` becomes `[FEATURE] Panel to configure X`.

4. **Validate the milestone.** Run:
   ```bash
   gh api repos/scub-france/Docling-Studio/milestones --jq '.[] | select(.state=="open") | .title'
   ```
   Abort if the provided milestone is not in the returned list. Show the open
   milestones so the user can pick.

5. **Resolve the label.** Run `gh label list --repo scub-france/Docling-Studio`.
   If the type has a matching label, use it. If not, show the open labels and
   ask the user which to apply. Do not guess. Known mapping at the time of
   writing: `feature` → `enhancement` (no `feature` label exists), all other
   types map 1:1.

6. **Load the repo template.** Read
   `.github/ISSUE_TEMPLATE/issue.md` from the local docling-studio checkout if
   available, otherwise fetch it from the repo. The issue body must preserve
   the template sections: Context · Scope · Current behavior · Expected
   behavior · Steps to reproduce / Acceptance criteria · Impact / Priority
   rationale · References.

7. **Fill the template.** If the user provided a body, slot it into the
   relevant sections. If only a title was provided, leave each section with a
   short `TODO:` placeholder so the assignee knows what to fill in — do not
   delete sections.

8. **Create the issue.** Use a heredoc for the body:
   ```bash
   gh issue create \
     --repo scub-france/Docling-Studio \
     --title "[TAG] Descriptive title" \
     --label "<resolved-label>" \
     --milestone "<milestone>" \
     --body "$(cat <<'EOF'
   <rendered template>
   EOF
   )"
   ```

9. **Report back.** Return the issue URL `gh` prints. Do not claim success
   unless `gh` exited 0.

## Refusals and recovery

- **Missing type**: ask which one; do not pick on the user's behalf.
- **Milestone not in the open list**: show the open list, ask the user to
  pick one or create it first (milestones are created outside this command).
- **Repo default branch or template file missing**: stop and report — do not
  fall back to a hand-rolled body.
- **`gh` not authenticated** (`gh auth status` fails): tell the user to run
  `gh auth login`; do not try to proceed.

## Why this command exists

See [`processes/issue-creation.md`](../../processes/issue-creation.md) in the
playbook for the rationale and the red flags this command prevents (untyped
issues, milestone-less tickets, off-template bodies).
