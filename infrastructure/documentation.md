# Documentation (MkDocs)

## Stack

- **MkDocs** with **Material** theme
- Deployed on **GitHub Pages** via GitHub Actions (`docs.yml`)

## Structure

```
docs/
  index.md                  # Home page
  getting-started.md        # Getting started guide
  architecture.md           # Architecture overview
  bbox-pipeline.md          # Bounding-box rendering pipeline notes
  claude-code-commands.md   # Reference of available /audit, /git, /ops, /release slash commands
  PROCESSES.md              # Process index
  contributing.md           # Contribution guide
  git-workflow/
    commit-conventions.md
    code-review-checklist.md
    merge-policy.md
  architecture/
    coding-standards.md
    adr-guide.md
    adr-template.md
    adrs/
      ADR-001-graph-visualization-library.md
  design/                   # Design docs (pre-ADR, or lightweight feature specs)
    neo4j-integration.md
    reasoning-trace.md
  release/
    deployment-checklist.md
    rollback-playbook.md
  operations/
    incident-response.md
    security-response.md
    monitoring-checklist.md
  community/
    onboarding-guide.md
    issue-triage-process.md
    roadmap-template.md
  audit/
    master.md               # Central orchestration
    audits/                 # 12 axis checklists (templates)
    reports/
      release-X.Y.Z/        # One folder per release, 12 reports + summary + remediation-plan
```

## Navigation (mkdocs.yml)

The site is organized into sections:
1. Home
2. Getting Started
3. Architecture
4. Contributing
5. Development (Coding Standards, Commits, Reviews, Merge, ADR)
6. Release (Deployment, Rollback)
7. Operations (Incident, Security, Monitoring)
8. Community (Onboarding, Issue Triage, Roadmap)
9. Audit (12 quality axes)

## Deployment

The `docs.yml` workflow automatically builds and deploys on push to `main`.

```bash
# Local preview
mkdocs serve

# Build
mkdocs build
```
