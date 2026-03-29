# Sonny AI CookBook

This project is a personal resource that collects guides and materials aimed at streamlining effective AI
development, configuring AI tools, and supporting agentic development.

I consider myself a sea dog of LLMs, and I have been exploring them since their public release. Surprisingly,
I am not a fan of vibecoding. I would rather stay an architect, design interfaces, choosing tooling, and
leverage the power of AI to extend my skills to ultimately ship production-ready code as it should!

> **Author's Note: READ BEFORE YOU CONTINUE**
>
> Everything you see here may not be best practice. Well, from a technical perspective it is, however, skills
> and prompts are based on my personal opinion, experience, and tuned on what works best for me... and may not
> work as you prefer for you!
>
> Remember that AI is based on transformers and not a real brain. These models are stochastic, and their output
> is hence non-deterministic and may do errors.

## The difference between README.md and AGENTS.md

[from the official AGENTS.md documentation](https://agents.md/):

README.md files are for humans: quick starts, project descriptions, and contribution guidelines.

AGENTS.md complements this by containing the extra, sometimes detailed context coding agents need: build steps,
tests, and conventions that might clutter a README or aren't relevant to human contributors.

We intentionally kept it separate to:

Give agents a clear, predictable place for instructions.

Keep READMEs concise and focused on human contributors.

Provide precise, agent-focused guidance that complements existing README and docs.

Rather than introducing another proprietary file, we chose a name and format that could work for anyone.
If you're building or using coding agents and find this helpful, feel free to adopt it.

## Quick Start

Browse the `guides/` directory for topic-specific guides. Each guide is a standalone markdown
file — pick what is relevant to your workflow and use it directly.

For Claude Code users, the `.claude/` directory contains ready-to-use skills you can drop into
your own projects.

## Contributing

Contributions are welcome. Keep the scope in mind: this is a documentation repository, so all
contributions should be markdown files (guides, skills, or configuration).

- Follow the conventions in [AGENTS.md](AGENTS.md).
- Lint your markdown with `markdownlint-cli2 "**/*.md"` before submitting.
- Open a pull request with a clear description of what you are adding or changing.

## What you can find here

- Markdowns! Lots of .md files in plain English
- Standard setup files for Claude Code, however I make my best to keep this setup vendor-agnostic,
  making it compatible or easily portable to other agents, such as GitHub Copilot or Cursor
- [AGENTS.md](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation):
  a simple, universal standard that gives AI coding agents a consistent source of project-specific guidance
  needed to operate reliably across different repositories and toolchains
- [CLAUDE.md](https://code.claude.com/docs/en/claude-directory): a special configuration file that lives in
  your repository and provides Claude with project-specific context. Note that to keep my setup portable and
  vendor-agnostic, CLAUDE.md simply points to AGENTS.md
- [.claude/skills](https://agentskills.io/home): Agent Skills are folders of instructions, scripts, and
  resources that agents can discover and use to do things more accurately and efficiently
