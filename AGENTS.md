# AGENTS.md

## What This Repository Is

A collection of guides and materials aimed at streamlining effective AI development, configuring
AI tools, and supporting agentic development.

This file is intended for coding agents and for human contributors who want the full detail behind
the rules and conventions. It complements README.md, which targets human readers with a quick
start and project overview.

## Repository Structure

This is a documentation repository. There are no build nor test. Users will pick what they need
to support and configure agentic development.

## How Skills Are Structured

`.claude/skills` are created with the best practices obtained from the
[official documentation](https://code.claude.com/docs/en/skills#create-your-first-skill).

Every `SKILL.md` must open with YAML frontmatter between `---` markers. Required fields:

```yaml
---
name: example-best-practices   # Slug for the /slash-command. Lowercase, hyphens, max 64.
description: One-line plain string. What it does and when to use it. Front-load the key case.
metadata:
  update-policy: monthly        # monthly | quarterly | manual | never
  update-source: https://example.org  # URL to fetch directly, or "web-search"
  last-updated: "2026-03-29"    # Date content was last refreshed: YYYY-MM-DD
---
```

The `metadata` key is used for project-specific fields. Do not place `update-policy`,
`update-source`, or `last-updated` at the top level — they are not part of the Claude Code
standard and will trigger validation warnings if placed outside `metadata`.

## Rules

README.md is for humans: quick start, project description, contribution guidelines. AGENTS.md
complements it with detailed context for coding agents and humans who want depth: conventions,
rules, and instructions that would clutter a README or are irrelevant to casual contributors.
Do not duplicate content between the two. If something belongs in README.md, it stays there.
If it belongs in AGENTS.md, it stays there.

- Whenever writing markdown, lint it using `markdownlint-cli2`.
- Whenever creating GitHub Actions, pin to a commit hash instead of a version tag.

## Core Concepts

**Role split:**

- Human = spec owner + evaluator (defines WHAT and WHY, never HOW)
- Powerful and expensive LLM = lead orchestrator (architecture decisions, spec review)
- Less powerful, and cheaper LLM = subagent worker

**Clean markdowns:**

- Plain english only
- max line length is 120
- linted markdown with respect to .markdownlint.jsonc, the same file used by the linter
- the linter is installed and available

## Lint Markdown

after writing each markdown file, run markdownlint-cli2 and fix any violations before moving to the next file:

```bash
markdownlint-cli2 "**/*.md"
```
