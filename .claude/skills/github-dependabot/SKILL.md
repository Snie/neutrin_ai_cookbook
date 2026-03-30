---
name: github-dependabot
description: Manages Dependabot alerts and PRs for a repository. Lists open security alerts grouped
  by severity and Dependabot PRs grouped by CI status. Surfaces everything for human review — never
  auto-merges.
metadata:
  update-policy: quarterly
  update-source: https://docs.github.com/en/rest/dependabot/alerts?apiVersion=2026-03-10
  last-updated: "2026-03-30"
---

# GitHub Dependabot: Alerts and Pull Requests

Load this skill whenever the user asks about Dependabot alerts, security vulnerabilities reported by
Dependabot, or open Dependabot pull requests.

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

## Step 1 — Fetch Security Alerts

Use the GitHub REST API via `gh api`. Pass the correct `Accept` and API-version headers.

```bash
gh api \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  "/repos/OWNER/REPO/dependabot/alerts?state=open&per_page=100"
```

API reference: <https://docs.github.com/en/rest/dependabot/alerts?apiVersion=2026-03-10#list-dependabot-alerts-for-a-repository>

Replace `OWNER/REPO` with the value derived in the previous step.

### Pagination

The endpoint is paginated. If the response contains 100 items, fetch subsequent pages by appending
`&page=2`, `&page=3`, … until a page returns fewer than 100 items.

### Display — Security Alerts

Present open alerts only, grouped by severity in descending order:

1. **Critical**
2. **High**
3. **Medium**
4. **Low**

For each alert, show:

| Field | Source path |
| ----- | ----------- |
| Alert number | `.number` |
| Package name | `.dependency.package.name` |
| Ecosystem | `.dependency.package.ecosystem` |
| Severity | `.security_vulnerability.severity` |
| CVE / GHSA | `.security_advisory.cve_id` or `.security_advisory.ghsa_id` |
| Patched version | `.security_vulnerability.first_patched_version.identifier` |
| URL | `.html_url` |

Example output format:

```text
## Security Alerts (open)

### Critical (2)
- #42  lodash (npm)  CVE-2021-23337  → patched in 4.17.21  https://github.com/…
- #38  axios (npm)   GHSA-…          → patched in 1.6.0    https://github.com/…

### High (1)
- #31  minimist (npm)  CVE-2021-44906  → patched in 1.2.6  https://github.com/…

### Medium (0)
### Low (0)
```

## Step 2 — Fetch Dependabot Pull Requests

List open PRs authored by the Dependabot app:

```bash
gh pr list --author "app/dependabot" --json number,title,url,statusCheckRollup,headRefName
```

CLI reference: <https://cli.github.com/manual/gh_pr_list>

### Display — Dependabot PRs

Group PRs by CI status:

**Passing** — all required checks green.
**Failing** — one or more required checks red or pending.

For each PR, show:

| Field | Description |
| ----- | ----------- |
| PR number | `#<number>` |
| Title | Full PR title |
| CI status | Passing / Failing / Pending |
| URL | PR URL |

Example output format:

```text
## Dependabot Pull Requests (open)

### CI Passing (1)
- #55  Bump actions/checkout from 4.2.2 to 4.3.0  https://github.com/…

### CI Failing / Pending (2)
- #54  Bump lodash from 4.17.20 to 4.17.21  https://github.com/…
- #51  Bump minimist from 1.2.5 to 1.2.6    https://github.com/…
```

## Output Order

Always present sections in this order:

1. Security Alerts (Step 1)
2. Dependabot PRs (Step 2)

Security alerts take priority — they appear first regardless of how many PRs exist.

## Rules

- **Never auto-merge** a Dependabot PR. Always surface PRs for human review.
- **Open alerts only** — do not display dismissed or fixed alerts unless the user explicitly asks.
- **No hardcoded owner/repo** — always infer from the git remote.
- **Abort on auth failure** — check `gh auth status` before all other commands.
- **Severity ordering** is always Critical → High → Medium → Low.
