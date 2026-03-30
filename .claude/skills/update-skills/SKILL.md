---
name: update-skills
description: Keeps all project skills current by checking update thresholds and fetching docs from each skill's update-source. Invoke as /update-skills, /update-skills --force, or /update-skills <skill-name>.
disable-model-invocation: true
metadata:
  update-policy: manual
  update-source: https://code.claude.com/docs/en/skills
  last-updated: "2026-03-29"
---

# Update Skills

Refresh skill content by fetching the authoritative source when the update threshold has passed.

## Invocation

```text
/update-skills                   # Respect update-policy thresholds
/update-skills --force           # Ignore last-updated, update everything
/update-skills <skill-name>      # Target one skill by name
```

## Requirements

### Skill metadata

Every `SKILL.md` in this project must open with YAML frontmatter between `---` markers.
In order to correctly work, this update-skills should have the following fields and valid values:

| Field | Required | Description |
| ----- | -------- | ----------- |
| `metadata` | Yes | Project-specific block. Not part of the Claude Code standard. |
| `metadata.update-policy` | Yes | `monthly` \| `quarterly` \| `manual` \| `never` |
| `metadata.update-source` | Yes | URL to fetch directly, or `web-search` for official docs search. |
| `metadata.last-updated` | Yes | Date content was last refreshed: `YYYY-MM-DD`. |

When not present, inform the user and ask him if and how he wants to set the required metadata.

### SKILL.md ideal setup

When a skill section is technical and contains commands and instructions, it should also contain at
 least a link to the official documentation or a valid web source for further readings and references.
 This part is up to the user, but this skill should inform him about this, and ask the user if he wants to
 enhance the mentioned sections with further readings and technical documentation URLs.

## Algorithm

**Step 1 — Discover skills**: Glob all `.claude/skills/*/SKILL.md`.

**Step 2 — Parse frontmatter**: Read `metadata.update-policy`, `metadata.update-source`, and
`metadata.last-updated` from each. In case this metadata is not present, warn the user and ask
him if and how to fill these fields

**Step 3 — Check threshold**: Compute `today - last-updated` in days.

| `update-policy` | Threshold | Behaviour |
| --------------- | --------- | --------- |
| `monthly` | 30 days | Update if delta ≥ 30. |
| `quarterly` | 90 days | Update if delta ≥ 90. |
| `manual` | — | Skip unless `--force` or targeted by name. |
| `never` | — | Always skip, even with `--force`. |

**Step 4 — Fetch source** for each skill that is stale or forced:

- If `update-source` is a URL → fetch that URL directly. Do not search for alternatives.
- If `update-source: web-search` → search for the official documentation only.
  Skip blogs, aggregators, and third-party tutorials.

**Step 5 — Update content**: Apply changes where the source differs from the current skill body.
Preserve all frontmatter fields. Do not remove sections that have no upstream equivalent.

**Step 6 — Bump `metadata.last-updated`**: Set to today's date (`YYYY-MM-DD`) after a successful update.

**Step 7 — Report**:

- Updated: list each skill that was refreshed.
- Skipped: list each skill that was within threshold or has `update-policy: manual/never`.
- Failed: list each skill where the fetch or parse failed, with the reason.

## Rules

- Never change `metadata.update-policy`, `metadata.update-source`, or `name` when updating a skill.
- Ideally, each section of a skill should contain a URL to the related part of the documentation
- Never update a skill with `metadata.update-policy: never`, even when `--force` is passed.
- When `metadata.update-source` is a URL, fetch it directly — do not search for it or substitute it.
- When `metadata.update-source: web-search`, use official documentation only. Prefer the tool's own
  docs site over any third-party source.
- Lint updated markdown with `markdownlint-cli2` before writing.
- Max line length is 120 characters.
