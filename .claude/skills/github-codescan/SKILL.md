---
name: github-codescan
description: Lists open code-scanning alerts for a repository, grouped by severity. Surfaces all
  findings for human review — never dismisses alerts automatically.
metadata:
  update-policy: quarterly
  update-source: https://docs.github.com/en/rest/code-scanning/code-scanning?apiVersion=2026-03-10
  last-updated: "2026-03-30"
---

# GitHub Code Scanning: Alerts

Load this skill whenever the user asks about code-scanning alerts, SAST findings, or CodeQL results
for a repository.

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

Store the result as `OWNER/REPO` and use it in every subsequent API call.

## Step 1 — Fetch Code Scanning Alerts

Use the GitHub REST API via `gh api`. Pass the correct `Accept` and API-version headers.

```bash
gh api \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  "/repos/OWNER/REPO/code-scanning/alerts?state=open&per_page=100"
```

API reference:
<https://docs.github.com/en/rest/code-scanning/code-scanning?apiVersion=2026-03-10#list-code-scanning-alerts-for-a-repository>

Replace `OWNER/REPO` with the value derived in the previous step.

### Pagination

The endpoint is paginated (max 100 per page). If the response contains 100 items, fetch subsequent
pages by appending `&page=2`, `&page=3`, … until a page returns fewer than 100 items.

### Display — Code Scanning Alerts

Present open alerts only, grouped by severity in descending order:

1. **Critical**
2. **High**
3. **Medium**
4. **Low**
5. **Warning** / **Note** / **Error** (catch-all for tool-specific levels)

For each alert, show:

| Field | Source path |
| ----- | ----------- |
| Alert number | `.number` |
| Rule ID | `.rule.id` |
| Rule description | `.rule.description` |
| Severity | `.rule.severity` |
| Tool | `.tool.name` |
| Location | `.most_recent_instance.location.path` + line `.most_recent_instance.location.start_line` |
| URL | `.html_url` |

Example output format:

```text
## Code Scanning Alerts (open)

### Critical (1)
- #12  js/code-injection  "Code injection"  [CodeQL]  src/server.js:42  https://github.com/…

### High (2)
- #9   js/xss             "Cross-site scripting"  [CodeQL]  src/render.js:17  https://github.com/…
- #7   js/path-injection  "Path injection"        [CodeQL]  src/upload.js:88  https://github.com/…

### Medium (0)
### Low (0)
### Warning / Note / Error (0)
```

## Output Order

Always present sections in severity order: Critical → High → Medium → Low → Warning/Note/Error.

## Rules

- **Never auto-dismiss** an alert. Always surface findings for human review.
- **Open alerts only** — do not display dismissed or fixed alerts unless the user explicitly asks.
- **No hardcoded owner/repo** — always infer from the git remote.
- **Abort on auth failure** — check `gh auth status` before all other commands.
- **Severity ordering** is always Critical → High → Medium → Low → Warning/Note/Error.
