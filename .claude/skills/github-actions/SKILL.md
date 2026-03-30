---
name: github-actions
description: Best practices for GitHub Actions in this docs-only repository. Load when creating or modifying .github/workflows/ files. Focus on pinning, path-filtered triggers, and linting.
metadata:
  update-policy: quarterly
  update-source: https://docs.github.com/en/actions
  last-updated: "2026-03-30"
---

# GitHub Actions: Documentation Repository Workflows

Apply these rules whenever you create or modify a file under `.github/workflows/`.

## Core Concepts

- **Workflow**: a YAML file in `.github/workflows/`. Runs on GitHub-hosted runners.
- **Trigger (`on`)**: the event that starts the workflow (push, pull_request, schedule, etc.).
- **Job**: a group of steps that run on the same runner.
- **Step**: a single shell command or a call to a reusable action (`uses`).
- **Action**: a reusable unit from the GitHub Actions marketplace or a local path.

## Security: Pin Actions to Commit Hashes

Never reference an action by a mutable tag (`@v4`) or branch (`@main`).
Always pin to the full 40-character commit SHA. Add the version tag as a comment.

```yaml
# Correct — pinned to commit hash
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2

# Wrong — mutable tag, vulnerable to supply chain attacks
- uses: actions/checkout@v4
```

Reason: tags can be moved or deleted by the upstream author. A commit hash is immutable.

## Triggers for Documentation Repositories

Use `paths` filters to avoid running workflows on unrelated changes.
This repository contains only markdown — filter to `**/*.md`:

```yaml
on:
  push:
    paths:
      - '**/*.md'
  pull_request:
    paths:
      - '**/*.md'
```

This ensures the workflow runs only when markdown files change.

## Workflow Structure

```yaml
name: <Descriptive Name>

on:
  push:
    paths:
      - '**/*.md'
  pull_request:
    paths:
      - '**/*.md'

jobs:
  <job-id>:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<commit-sha>  # v4.x.x
      - uses: <action>@<commit-sha>          # vX.Y.Z
        with:
          <key>: <value>
```

For further details refer to
<https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#permissions>

## Secrets and Sensitive Data

- Store sensitive values as repository, environment, or organisation secrets.
- Reference them with `${{ secrets.MY_SECRET }}` — never hardcode credentials.
- The `GITHUB_TOKEN` is automatically provided; use it for repo-scoped operations.

## Best Practices for This Repository

- No build or test jobs. Scope workflows to linting and validation only.
- Use `ubuntu-latest` for runner; no special requirements for markdown processing.
- Keep each workflow focused on one concern (one workflow per tool or task).
- Use `name:` fields on steps to make the Actions log easy to read.
- Avoid `workflow_dispatch` unless you have a specific manual-run requirement.
- Use `concurrency` groups to cancel redundant runs on the same branch:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## Recommended Actions for This Repository

| Action | Purpose | Current pinned version |
| ------ | ------- | ---------------------- |
| `actions/checkout` | Check out the repository | v6.0.2 |
| `DavidAnson/markdownlint-cli2-action` | Lint markdown files | v23.0.0 |

Always verify the commit hash before use. Fetch the latest tag from the action's repository
and confirm the SHA with the GitHub API before updating pinned hashes.
