---
name: readme-agents-sync
description: Checks README.md and AGENTS.md for consistency — correct scope, no duplication, matching project description, and a visible pointer from README.md to AGENTS.md.
metadata:
  update-policy: manual
  update-source: web-search
  last-updated: "2026-03-29"
---

# README and AGENTS Sync

Run this skill whenever `README.md` or `AGENTS.md` is modified.

## What Each File Must Cover

### README.md — for humans

- What the repository is (project description, purpose).
- Repository structure at a high level.
- Quick start: how to get started using the materials.
- Contribution guidelines: how to contribute.
- A visible pointer to `AGENTS.md` for contributors who want full detail.

### AGENTS.md — for agents and detail-oriented humans

- Rules and conventions that govern how work is done in this repository.
- Instructions for coding agents (build, test, lint commands, workflow).
- Context that would clutter README.md or is irrelevant to casual contributors.
- The authoritative frontmatter schema for `.claude/skills/*/SKILL.md`.
- Details about the AGENTS.md standard can be found at [agents.md](https://agents.md/)

## Checklist

Run through every item. Flag any that fail.

1. **Scope — README.md**: Confirm it covers quick start, structure, and contribution guidelines.
   It must not contain agent instructions, rules, or conventions that belong in AGENTS.md.

2. **Scope — AGENTS.md**: Confirm it covers rules, conventions, and agent instructions.
   It must not duplicate content already in README.md, but rather extend it maintaining its scope.

3. **No duplication**: Check that no paragraph, rule, or section appears in both files.
   Summaries and cross-references are allowed; duplicated content is not.

4. **Consistent project description**: The first paragraph of README.md and the first paragraph
   of `## What This Repository Is` in AGENTS.md must describe the same project in consistent
   terms. They do not need to be identical, but they must not contradict each other.

5. **Pointer in README.md**: README.md must contain a visible link or reference to `AGENTS.md`
   so contributors know where to find the full detail. A sentence like "See [AGENTS.md](AGENTS.md)
   for rules and conventions" satisfies this requirement.

## Output

Report the result of each checklist item:

- Pass: item is satisfied.
- Fail: item is not satisfied. State what is wrong and suggest the fix.
- Warning: item is partially satisfied or ambiguous.

If all items pass, confirm that README.md and AGENTS.md are in sync.
If any item fails, list the required changes and ask for confirmation before editing.
