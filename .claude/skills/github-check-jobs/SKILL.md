---
name: github-check-jobs
description: Lists failing CI runs and their failed job steps for a repository. Surfaces failures
  grouped by workflow for human review — never retries or cancels runs automatically.
metadata:
  update-policy: quarterly
  update-source: https://cli.github.com/manual/gh_run_view
  last-updated: "2026-03-30"
---

# GitHub Check Jobs: Failing CI Runs

Load this skill whenever the user asks about failing CI jobs, broken workflows, or run failures on
the remote.

## Prerequisites

- **GitHub CLI** must be installed. See <https://github.com/cli/cli#installation>.
- **Authentication**: before any other step, run the command below and abort with a clear message if
  it fails.

```bash
gh auth status
```

If the command exits non-zero, stop and tell the user:

> GitHub CLI is not authenticated. Run `gh auth login` and try again.

## Infer OWNER and REPO

Never hardcode the repository owner or name. Derive them from the current git remote:

```bash
gh repo view --json owner,name --jq '"\(.owner.login)/\(.name)"'
```

Store the result as `OWNER/REPO` and use it in every subsequent command.

## Step 1 — List Recent Failing Runs

Fetch the most recent runs that did not succeed. Limit to the last 20 to keep output manageable.

```bash
gh run list \
  --repo OWNER/REPO \
  --status failure \
  --limit 20 \
  --json databaseId,displayTitle,workflowName,status,conclusion,headBranch,createdAt,url
```

CLI reference: <https://cli.github.com/manual/gh_run_list>

Also fetch runs with conclusion `startup_failure` or `timed_out` by repeating the command with
`--status startup_failure` and `--status timed_out`. Merge all results before displaying.

### Display — Failing Runs

Group results by `workflowName`. For each run show:

| Field | Source |
| ----- | ------ |
| Run ID | `.databaseId` |
| Title | `.displayTitle` |
| Branch | `.headBranch` |
| Conclusion | `.conclusion` |
| Created at | `.createdAt` |
| URL | `.url` |

Example output format:

```text
## Failing CI Runs

### Lint Markdown (2)
- #1234  "Bump actions/checkout"  main  failure  2026-03-30  https://github.com/…
- #1230  "Fix typo in README"     main  failure  2026-03-29  https://github.com/…

### CodeQL (1)
- #1228  "first commit"  main  timed_out  2026-03-28  https://github.com/…
```

## Step 2 — Inspect Failed Steps for Each Run

For each run found in Step 1, fetch the failed step logs using the run ID:

```bash
gh run view RUN_ID --repo OWNER/REPO --log-failed
```

CLI reference: <https://cli.github.com/manual/gh_run_view>

If there are more than 5 failing runs, inspect only the 5 most recent to avoid excessive output.
Inform the user if runs were skipped.

### Display — Failed Steps

Under each run, list the steps that failed with their log excerpt (first 20 lines per step):

```text
### Run #1234 — "Bump actions/checkout" (Lint Markdown / main)

Step: Lint markdown
<log excerpt>
```

## Output Order

Always present sections in this order:

1. Failing Runs summary (Step 1)
2. Failed step details (Step 2), in the same run order as Step 1

## Rules

- **Never auto-retry or cancel** a run. Always surface failures for human review.
- **No hardcoded owner/repo** — always infer from the git remote.
- **Abort on auth failure** — check `gh auth status` before all other commands.
- **Cap detailed inspection at 5 runs** — warn the user if more were found but skipped.
- **Most recent first** — always sort runs by `createdAt` descending.
