# Spec-Driven Swarm Development
## A Practical Guide for the Budget-Conscious AI Architect

> **Role split**: You own the spec and interface contracts. Agents own execution. Claude orchestrates.  
> **Budget constraint**: Foundation model APIs only. No local GPU cluster.  
> **Target**: Repeatable, parallelized website (and beyond) factory.

---

## Table of Contents

1. [Core Philosophy](#1-core-philosophy)
2. [Toolchain](#2-toolchain)
3. [Project Structure](#3-project-structure)
4. [Layer 0 — The Spec (Your Job)](#4-layer-0--the-spec-your-job)
5. [Layer 1 — Interface Contracts (Blocking Gate)](#5-layer-1--interface-contracts-blocking-gate)
6. [Layer 2 — Token Architecture (Don't Burn Money)](#6-layer-2--token-architecture-dont-burn-money)
7. [Layer 3 — Agent Definitions](#7-layer-3--agent-definitions)
8. [Layer 4 — Orchestration Flow](#8-layer-4--orchestration-flow)
9. [Layer 5 — Docker Sandbox Execution](#9-layer-5--docker-sandbox-execution)
10. [Layer 6 — Context Survival (PreCompact)](#10-layer-6--context-survival-precompact)
11. [The Mitko Vasilev Approach — What's Feasible on a Budget](#11-the-mitko-vasilev-approach--whats-feasible-on-a-budget)
12. [Running a Project End-to-End](#12-running-a-project-end-to-end)
13. [Reference Cheatsheet](#13-reference-cheatsheet)

---

## 1. Core Philosophy

```
Spec quality   →  the real bottleneck, not compute
Interface contracts  →  generated FIRST, never invented by agents
Subagents      →  one context window per module, isolated
You            →  architect + evaluator, not coder
Claude         →  lead orchestrator on Opus, workers on Sonnet
Token cost     →  controlled by structure, not luck
```

**The one rule that prevents all disasters:**  
Agents consume interfaces. They never invent them.  
If an agent is deciding what an API endpoint looks like, the process is already broken.

---

## 2. Toolchain

| Tool | Purpose | Cost |
|---|---|---|
| `spec-kit` (GitHub) | Structured spec lifecycle: `/specify → /plan → /tasks → /implement` | Free |
| Claude Code | Lead orchestrator + subagent spawner | API cost |
| Docker Sandboxes | Isolation per agent, pre-baked images | Docker Desktop |
| `ccusage` | Live token burn monitoring, 5-hour billing windows | Free |
| `awesome-agent-skills` | Community SKILL.md library, 1000+ files, lazy-loaded | Free |
| `DSPy` | GEPA skill self-optimization loop (see Section 11) | Free (your API cost) |

### Bootstrap any project

```bash
# Spec lifecycle scaffolding
uvx --from git+https://github.com/github/spec-kit.git specify init my-project \
  --ai claude --ai-skills

# Install skills selectively (don't load all — lazy loading is the point)
npx antigravity-awesome-skills --claude

# Token monitoring
pip install ccusage
ccusage blocks --live
```

---

## 3. Project Structure

```
project-root/
│
├── CLAUDE.md                        # ≤200 lines. INDEX ONLY. No content.
│
├── .claude/
│   ├── agents/                      # Persistent specialist agent definitions
│   │   ├── backend-agent.md
│   │   ├── frontend-agent.md
│   │   ├── design-agent.md
│   │   └── integration-agent.md
│   ├── rules/                       # One concern per file, auto-loaded
│   │   ├── coding-principles.md
│   │   ├── testing.md
│   │   ├── git.md
│   │   └── security.md
│   ├── commands/                    # spec-kit slash commands
│   ├── hooks/
│   │   └── precompact.md            # Context survival instructions
│   └── skills/                      # Lazy-loaded SKILL.md files
│       ├── nextjs.md
│       ├── tailwind.md
│       └── prisma.md
│
├── docs/
│   ├── spec.md                      # The contract. WHAT + WHY. Never HOW.
│   ├── interface-contracts/         # Generated FIRST. Agents read, never write.
│   │   ├── api.yaml                 # OpenAPI schema
│   │   ├── db-schema.sql            # Database schema
│   │   ├── types.ts                 # Shared TypeScript types
│   │   └── component-api.md        # Component prop contracts
│   ├── tasks/                       # Per-agent scoped task files
│   │   ├── task-backend.md
│   │   ├── task-frontend.md
│   │   ├── task-design.md
│   │   └── task-infra.md
│   ├── decisions/                   # ADRs — survive context resets
│   │   └── adr-YYYY-MM-DD.md
│   └── blockers.md                  # Agents append here, you resolve
│
├── backend/
│   ├── CLAUDE.md                    # Loaded ON DEMAND only (saves tokens)
│   └── ...
├── frontend/
│   ├── CLAUDE.md                    # Loaded ON DEMAND only
│   └── ...
└── infra/
    ├── CLAUDE.md                    # Loaded ON DEMAND only
    └── ...
```

### Why subdirectory CLAUDE.md files matter for cost

Subdirectory CLAUDE.md files are loaded on-demand, not at startup. When the backend agent runs, only `backend/CLAUDE.md` loads. Frontend rules cost zero tokens for the backend session. At scale across multiple agents and projects, this compounds.

### Root CLAUDE.md template

```markdown
# [Project Name] — Claude Orchestration Index

## Tech Stack
- Frontend: [e.g. Next.js 15, Tailwind, shadcn/ui]
- Backend: [e.g. Node/Fastify, Prisma, PostgreSQL]
- Infra: [e.g. Docker Compose, Fly.io]

## Domain Boundaries
- `/backend` → backend-agent owns it
- `/frontend` → frontend-agent owns it  
- `/infra` → infra-agent owns it
- `/docs/interface-contracts` → READ ONLY for all agents

## Parallelization Rules
When building, spawn parallel subagents by default.
Phase 1: backend + design + infra (independent)
Phase 2: frontend (depends on Phase 1 contracts)
Phase 3: integration review (sequential)

## Key Files
- Spec: @docs/spec.md
- API contract: @docs/interface-contracts/api.yaml
- Types: @docs/interface-contracts/types.ts
- Blockers: @docs/blockers.md

## Submodule Context
- Backend details: @backend/CLAUDE.md (load on demand)
- Frontend details: @frontend/CLAUDE.md (load on demand)
```

---

## 4. Layer 0 — The Spec (Your Job)

The spec is the system. Everything downstream is mechanical.

### What belongs in spec.md

```markdown
# Project Spec: [Name]

## Purpose
[What this does and why. 2-3 sentences max.]

## Users & Key Flows
- [User type A] needs to [do X] so that [outcome]
- [User type B] needs to [do Y] so that [outcome]

## Pages / Routes
| Route | Purpose | Auth required |
|---|---|---|
| / | Landing page | No |
| /dashboard | Main app | Yes |
| /api/v1/* | REST API | Yes (JWT) |

## Data Model (conceptual, not SQL)
- User: id, email, name, role, created_at
- Project: id, owner_id, name, status, config (JSON)
- ...

## Business Rules
- [Concrete constraints the code must enforce]
- [Validation rules]
- [Access control logic]

## Out of Scope
- [Explicit exclusions to prevent agent scope creep]
```

### What does NOT belong in spec.md

- Implementation choices (which library, which pattern)
- File structure decisions
- How to handle edge cases not specified in business rules

Anything not in the spec is an agent's call. If you care about it, specify it.

### Spec lifecycle with spec-kit

```bash
# Inside Claude Code session:
/specify     # Claude refines your raw notes into structured spec
/plan        # Claude generates technical plan — YOU REVIEW + CORRECT
/tasks       # Claude decomposes into per-agent task files → docs/tasks/
/implement   # Kicks off execution
```

---

## 5. Layer 1 — Interface Contracts (Blocking Gate)

**This runs BEFORE any parallel agents start. It is sequential and non-negotiable.**

Generate contracts in one Claude Code session:

```
Generate interface contracts for @docs/spec.md.

Produce the following files:
1. docs/interface-contracts/api.yaml        — OpenAPI 3.1, all endpoints, request/response schemas
2. docs/interface-contracts/db-schema.sql   — Full schema, indexes, foreign keys
3. docs/interface-contracts/types.ts        — All shared TypeScript interfaces
4. docs/interface-contracts/component-api.md — Each component: name, props, variants, usage

Do not generate implementation. Contracts only.
Review each file and flag any ambiguity before proceeding.
```

**You review the output. You correct anything architecturally wrong.**  
This is your primary leverage point. 30 minutes here saves hours of merge conflicts.

---

## 6. Layer 2 — Token Architecture (Don't Burn Money)

### The cost math

| Model | Input $/1M | Output $/1M | Output multiplier |
|---|---|---|---|
| Claude Opus 4 | $15 | $75 | 5× |
| Claude Sonnet 4.6 | $3 | $15 | 5× |
| Claude Haiku 4.5 | $0.80 | $4 | 5× |

Output tokens are always 5× input cost. Everything that reduces output volume, wins.

### The five levers, ranked by ROI

**1. Plan mode by default — ~50% cost reduction**

```bash
# In Claude Code
Shift+Tab    # toggle plan mode
# Or prefix any prompt:
!plan analyze the auth module and propose the testing strategy
```

Use plan mode for: analysis, architecture decisions, reviews, debugging strategies.  
Exit plan mode only when you need file modifications.

**2. Opus plans, Sonnet executes — ~80% cost reduction on execution**

```bash
# In ~/.bashrc or ~/.zshrc
alias opusplan='ANTHROPIC_MODEL=claude-opus-4-6 claude --plan'

# Workflow:
opusplan    # architect-level reasoning on complex decisions
# then switch back to default Sonnet for implementation
```

Set subagent model explicitly:
```bash
export CLAUDE_CODE_SUBAGENT_MODEL=claude-sonnet-4-6
```

**3. Skills over inline context — ~30% startup cost reduction**

Bad pattern (bloated CLAUDE.md):
```markdown
## Next.js Rules
When using Next.js, always use the App Router. Route files should be named page.tsx.
Server components are default. Use 'use client' only when needed. Always use...
[200 more lines]
```

Good pattern (lazy skill):
```markdown
## Skills Available
- Next.js patterns: @.claude/skills/nextjs.md (load when working in /frontend)
```

The skill body only loads when triggered. 200 lines × N agents × M sessions = significant.

**4. Disable unused MCP servers per task**

Each enabled MCP server adds tool definitions to your system prompt.

```bash
# Inside Claude Code session
/mcp         # see active servers
# Disable what's irrelevant for current task
```

For a pure code-generation session: disable calendar, email, drive MCP servers.

**5. Environment flag for background model calls**

```bash
export DISABLE_NON_ESSENTIAL_MODEL_CALLS=1
```

Kills background ambient calls Claude Code makes. No visible impact on core functionality.

### Monitoring

```bash
ccusage blocks --live    # 5-hour billing block, live burn rate
# Inside Claude Code:
/context    # breakdown: system / tools / memory / skills / messages
/cost       # running session total
```

**Set a mental budget per project before starting.** Without it, you won't know when a runaway agent session is burning $40 in 20 minutes.

---

## 7. Layer 3 — Agent Definitions

Each specialist lives in `.claude/agents/[name].md`.

### Backend agent example

```yaml
---
name: backend-agent
description: >
  Owns all backend code: API routes, auth, business logic, database access.
  Invoke for any task touching /backend. Reads interface contracts, never modifies them.
model: claude-sonnet-4-6
tools: [read, write, bash, git]
---

## Scope
- Owns: /backend/**
- Reads (never writes): /docs/interface-contracts/api.yaml, /docs/interface-contracts/db-schema.sql, /docs/interface-contracts/types.ts
- Never touches: /frontend, /infra

## Constraints
- Implement exactly to @docs/interface-contracts/api.yaml — no endpoint invention
- All business logic must have unit tests (vitest)
- No hardcoded secrets — use process.env with validation on startup

## Done Criteria
- All routes from api.yaml implemented
- npm run test passes with >80% coverage
- npm run build succeeds
- Committed: feat(backend): [description]
- Blockers logged to @docs/blockers.md if any
```

### Frontend agent example

```yaml
---
name: frontend-agent  
description: >
  Owns all frontend code: pages, components, routing, styling.
  Invoked AFTER design-agent has generated the design system.
  Reads component API contracts and API schema.
model: claude-sonnet-4-6
tools: [read, write, bash, git]
---

## Scope
- Owns: /frontend/**
- Reads: /docs/interface-contracts/api.yaml, /docs/interface-contracts/component-api.md, /docs/interface-contracts/types.ts
- Consumes design tokens from: /frontend/design-system/tokens/

## Constraints
- Use components from design system only — no inline CSS, no ad-hoc styling
- API calls must match api.yaml exactly — no invented endpoints
- All pages must be responsive (mobile-first)

## Done Criteria
- All routes from spec implemented
- npm run build succeeds
- npm run test passes
- Committed: feat(frontend): [description]
```

### Design agent example

```yaml
---
name: design-agent
description: >
  Owns design system: tokens, base components, Storybook.
  Runs in Phase 1 (parallel with backend/infra).
  Frontend agent depends on its output.
model: claude-sonnet-4-6
tools: [read, write, bash]
---

## Scope
- Owns: /frontend/design-system/**
- Reads: /docs/interface-contracts/component-api.md

## Output Contract (frontend-agent depends on this)
Produce the following before finishing:
- /frontend/design-system/tokens/colors.ts
- /frontend/design-system/tokens/typography.ts
- /frontend/design-system/tokens/spacing.ts
- /frontend/design-system/components/[each component in component-api.md]

## Done Criteria
- All components from component-api.md implemented
- Each component: typed props, default export, Tailwind only
- npm run storybook builds without error
- Committed: feat(design): [description]
```

---

## 8. Layer 4 — Orchestration Flow

### The master orchestration prompt

```
You are the lead orchestrator for this project.

Read: @docs/spec.md
Read: @docs/interface-contracts/ (all files)
Read: @CLAUDE.md

Execute in phases. Use the Task tool to spawn subagents. Do not wait for one phase 
to complete before spawning all tasks in that phase simultaneously.

---

PHASE 1 — PARALLEL (no shared state, run simultaneously):

Task A — Backend:
  Agent: backend-agent
  Task: @docs/tasks/task-backend.md
  Workspace: /backend
  
Task B — Design System:
  Agent: design-agent  
  Task: @docs/tasks/task-design.md
  Workspace: /frontend/design-system

Task C — Infrastructure:
  Agent: infra-agent
  Task: @docs/tasks/task-infra.md
  Workspace: /infra

---

PHASE 2 — PARALLEL (after Phase 1 commits):

Task D — Frontend:
  Agent: frontend-agent
  Task: @docs/tasks/task-frontend.md
  Workspace: /frontend
  Depends on: Phase 1 Task B output (design tokens + components)

---

PHASE 3 — SEQUENTIAL (after all Phase 2 commits):

Task E — Integration review:
  Read all outputs.
  Run full test suite.
  Check API contract compliance.
  Log conflicts to @docs/blockers.md.
  Produce final status report.

---

If any agent hits a blocker it cannot resolve, it must:
1. Log to @docs/blockers.md with full context
2. Commit work-in-progress with prefix wip(module):
3. Return control

Do not invent solutions to blockers. Surface them.
```

### Per-agent task file format

`docs/tasks/task-backend.md`:
```markdown
# Backend Task

## Context
- Full spec: @docs/spec.md (sections: Data Model, Business Rules, Routes)
- API contract to implement: @docs/interface-contracts/api.yaml
- DB schema: @docs/interface-contracts/db-schema.sql
- Shared types: @docs/interface-contracts/types.ts

## Your Job
Implement the backend API exactly as specified in api.yaml.

Do NOT:
- Invent endpoints not in api.yaml
- Modify interface-contracts/ files
- Touch /frontend or /infra

## Stack
- Node.js + Fastify
- Prisma ORM (schema in db-schema.sql)
- JWT auth via @fastify/jwt
- Vitest for tests

## Acceptance
- [ ] All routes from api.yaml implemented
- [ ] Auth middleware on protected routes
- [ ] Unit tests for all business logic
- [ ] npm run test passes
- [ ] npm run build passes
- [ ] Committed with feat(backend): prefix
```

---

## 9. Layer 5 — Docker Sandbox Execution

### Setup

```bash
# In ~/.zshrc (must be set BEFORE Docker Desktop starts)
export ANTHROPIC_API_KEY=sk-ant-api03-xxxxx

# Source and restart Docker Desktop after adding
source ~/.zshrc
```

### Per-project sandbox creation

```bash
# Create named sandboxes per module
docker sandbox create claude ~/project/backend  --name proj-backend
docker sandbox create claude ~/project/frontend --name proj-frontend
docker sandbox create claude ~/project/infra    --name proj-infra
```

### Execution patterns

**Manual parallel (simple shell):**
```bash
#!/bin/bash
# Phase 1 — parallel
docker sandbox run proj-backend  -- "$(cat docs/tasks/task-backend.md)"  &
docker sandbox run proj-infra    -- "$(cat docs/tasks/task-infra.md)"    &
docker sandbox run proj-design   -- "$(cat docs/tasks/task-design.md)"   &
wait

echo "Phase 1 complete. Starting Phase 2..."

# Phase 2 — frontend depends on design output
docker sandbox run proj-frontend -- "$(cat docs/tasks/task-frontend.md)"
```

**Python DAG (when you need dependency ordering + retries):**
```python
import subprocess
import concurrent.futures

tasks = {
    "backend":  ("proj-backend",  "docs/tasks/task-backend.md"),
    "design":   ("proj-design",   "docs/tasks/task-design.md"),
    "infra":    ("proj-infra",    "docs/tasks/task-infra.md"),
}

def run_sandbox(name, task_file):
    prompt = open(task_file).read()
    result = subprocess.run(
        ["docker", "sandbox", "run", name, "--", prompt],
        capture_output=True, text=True
    )
    print(f"[{name}] exit={result.returncode}")
    return result.returncode == 0

# Phase 1 — parallel
with concurrent.futures.ThreadPoolExecutor() as ex:
    futures = {ex.submit(run_sandbox, s, f): k 
               for k, (s, f) in tasks.items()}
    results = {k: f.result() for f, k in futures.items()}

if all(results.values()):
    # Phase 2
    run_sandbox("proj-frontend", "docs/tasks/task-frontend.md")
else:
    print("Phase 1 failures:", [k for k,v in results.items() if not v])
```

### Custom sandbox image (recommended for web projects)

Pre-install your stack so agents can run and test code, not just write it:

```dockerfile
FROM docker/sandbox-templates:claude-code

# Node.js stack
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
RUN apt-get install -y nodejs
RUN npm install -g pnpm typescript

# Your framework-specific tools
RUN npm install -g @prisma/cli vitest

WORKDIR /workspace
```

An agent that can `npm run test` and fix the failure is worth 10x an agent that only writes files.

---

## 10. Layer 6 — Context Survival (PreCompact)

Compaction fires at ~83.5% context fill (hardcoded). Standard compaction loses session state. Your defense:

### `.claude/hooks/precompact.md`

```markdown
# PreCompact Instructions

Before context compaction, preserve the following by appending to relevant files:

## 1. Decisions made this session
Append to docs/decisions/adr-{today's date}.md:
- Any architectural decisions made
- Any interface changes agreed
- Any spec ambiguities resolved

## 2. Current task status
Update docs/tasks/task-[module].md:
- Mark completed items with [x]
- Add NOTE: [what was completed before compaction]
- Add RESUME: [exact next step]

## 3. Active blockers
Append to docs/blockers.md:
- Any unresolved issues with full context

## 4. Session cost snapshot
Log current /cost output to docs/decisions/costs.md

These files survive compaction. The next session reads them to resume.
```

### Mid-session preservation habit

Any time you make a significant architectural call:
```
Document this decision in docs/decisions/adr-{today}.md so it survives 
context reset: [the decision and its rationale]
```

This is cheap insurance. Do it reflexively.

---

## 11. The Mitko Vasilev Approach — What's Feasible on a Budget

Mitko's full stack requires a 4× A6000 GPU workstation running 20 concurrent local agents at zero marginal token cost. That's off the table. Here's what translates.

### What he's actually doing (the ideas, not the hardware)

**GEPA: Skills as tunable parameters**

The key insight: a SKILL.md file is a prompt that can be optimized. Instead of manually rewriting it when things go wrong, you run a feedback loop:

```
SKILL.md (v1) → agent executes task → pass/fail/logs → improved SKILL.md (v2) → repeat
```

The optimization algorithm (GEPA, Genetic-Pareto) maintains a Pareto frontier of skill variants — one might be great at database tasks but mediocre at API design, another the inverse. A merge proposer then combines complementary strengths.

His published results:
- Claude Haiku task pass rate: 79% → 98% on repository tasks  
- ARC-AGI agent: 32% → 89% accuracy after 50 iterations
- Same model. No fine-tuning. Better instructions.

**The local swarm concept**  
20 concurrent agents, each with isolated context, coordinating through Git issues as the task queue. No chat UI — the issue tracker IS the orchestration surface.

**Radicle as agent UI**  
Distributed Git with issues. Agents consume issues as tasks, commit results, close issues. Human reviews the queue and approves merges.

---

### What you can run on API budget

**1. Manual GEPA-lite: skill evolution with DSPy**

You don't need local GPU for GEPA. DSPy calls the API.

```python
import dspy

# Your skill to optimize
skill_v1 = open(".claude/skills/nextjs.md").read()

# Your evaluator (THIS IS WHAT YOU DEFINE — the bottleneck)
def evaluator(skill_text: str) -> float:
    """
    Run a set of real tasks with this skill, measure pass rate.
    Returns 0.0–1.0
    """
    # Replace with your actual test suite runner
    tasks = load_test_tasks()  # real tasks from your projects
    results = []
    for task in tasks:
        prompt = f"Using these guidelines:\n{skill_text}\n\nTask: {task.description}"
        response = call_claude_sonnet(prompt)  # cheap model for eval
        results.append(task.evaluate(response))
    return sum(results) / len(results)

# Run optimization
# DSPy handles the Pareto variant generation
optimizer = dspy.teleprompt.MIPRO(metric=evaluator)
optimized_skill = optimizer.compile(skill_v1)
open(".claude/skills/nextjs-v2.md", "w").write(optimized_skill)
```

**Cost control**: run GEPA on Haiku ($0.80/1M input). The skill improvements transfer to Sonnet and Opus sessions. Amortized cost per project drops as your skill library matures.

**2. Git Issues as agent queue (GitHub, not Radicle)**

```markdown
# Issue template: Agent Task
**Agent**: backend-agent  
**Phase**: 1  
**Task file**: docs/tasks/task-backend.md  
**Depends on**: none  
**Status**: ready
```

```bash
# Minimal orchestrator reading GitHub Issues
gh issue list --label "agent-task" --label "phase-1" --json number,title,body \
  | jq -r '.[] | "\(.number) \(.title)"'

# Agent picks up issue, runs, closes it with commit reference
gh issue close 42 --comment "Completed in commit abc1234. Tests passing."
```

Your human workflow: scan open issues → review closed ones with failing tests → reopen with clarification.

**3. Skill specialization by project type**

Build a skill library that improves across projects, not just within one:

```
.claude/skills/
├── stacks/
│   ├── nextjs-app-router.md        # refined over 5 projects
│   ├── fastify-prisma.md           # refined over 3 projects
│   └── tailwind-shadcn.md          # refined over 8 projects
├── patterns/
│   ├── auth-jwt.md
│   ├── stripe-integration.md
│   └── s3-uploads.md
└── evaluators/
    └── test-tasks/                  # real tasks for GEPA evaluation
        ├── auth-task.md
        ├── crud-api-task.md
        └── responsive-layout-task.md
```

Every project refines the library. By project 10, your agents are measurably better than project 1 with no prompt engineering effort per-project.

**4. The evaluator is your real leverage**

Mitko's warning (and it's correct): GEPA optimizes against whatever you measure. Bad evaluator → agents pass tests while producing unmaintainable code.

Good evaluators for web projects:
```python
def composite_evaluator(output_dir: str) -> float:
    scores = []
    
    # Hard correctness (binary)
    scores.append(1.0 if run_tests(output_dir) else 0.0)
    scores.append(1.0 if build_succeeds(output_dir) else 0.0)
    scores.append(1.0 if api_contract_compliant(output_dir) else 0.0)
    
    # Soft quality (gradients)
    scores.append(type_coverage(output_dir))        # 0.0–1.0
    scores.append(test_coverage(output_dir))         # 0.0–1.0
    scores.append(1.0 - dead_code_ratio(output_dir)) # 0.0–1.0
    
    # YOU review skill diffs after >50 iterations
    # Reject cargo-cult accumulation
    return sum(scores) / len(scores)
```

After ~50 GEPA iterations, review the skill diff manually. Skill files can accumulate contradictory rules that happen to work until the codebase changes.

---

### The Budget GEPA Cadence

```
Week 1–2:  Build first project manually. Note what agents got wrong.
Week 3:    Define evaluator test tasks from real failures.
Week 4:    Run GEPA on Haiku against your 5 worst-performing skills.
           Cost: ~$2–5 per skill optimization run.
Month 2+:  Each new project runs with better skills.
           Review skill diffs. Prune contradictions.
           Repeat GEPA quarterly or after 3 new project types.
```

---

## 12. Running a Project End-to-End

### Checklist

```
SETUP (once per project type, then reuse)
[ ] Stack decided and written into CLAUDE.md
[ ] Agent definitions in .claude/agents/ (or reused from library)
[ ] Skills curated in .claude/skills/ (lazy-load references in CLAUDE.md)
[ ] Docker sandbox images built with your stack pre-installed

SPEC PHASE (you, ~1-2 hours)
[ ] spec.md written (what + why, no how)
[ ] /specify run — Claude refines
[ ] /plan run — you review and correct architecture
[ ] Interface contracts generated (sequential, blocking)
[ ] You review and approve contracts
[ ] /tasks run — Claude generates per-agent task files

EXECUTION PHASE
[ ] Phase 1 sandboxes launched (parallel): backend + design + infra
[ ] Phase 2 sandbox launched (after Phase 1 commits): frontend
[ ] Phase 3: integration review agent (sequential)
[ ] Blockers in docs/blockers.md reviewed and resolved by you

EVALUATION (feeds GEPA loop)
[ ] Test suite run — note failure patterns
[ ] Pass/fail logged against skill versions used
[ ] If pattern repeats across 2+ projects: trigger GEPA on that skill
```

---

## 13. Reference Cheatsheet

### Token cost reduction by technique

| Technique | Est. savings | Effort |
|---|---|---|
| Plan mode on analysis tasks | ~50% | Zero |
| Sonnet for subagents (vs Opus) | ~80% execution cost | One env var |
| Lazy skills (vs inline CLAUDE.md) | ~30% startup | Restructure once |
| Disable unused MCP servers | ~5-15% | Per session |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS=1` | ~5% | One env var |
| PreCompact hooks (avoid re-doing work) | Hard to quantify but real | Set up once |

### Quick commands

```bash
# Monitor
ccusage blocks --live
# In Claude Code session:
/context
/cost
/mcp           # toggle MCP servers

# Orchestration
export CLAUDE_CODE_SUBAGENT_MODEL=claude-sonnet-4-6
export DISABLE_NON_ESSENTIAL_MODEL_CALLS=1
export ANTHROPIC_MODEL=claude-opus-4-6  # for planning session

# Sandboxes
docker sandbox create claude ~/project/backend --name proj-backend
docker sandbox run proj-backend -- "$(cat docs/tasks/task-backend.md)"

# GEPA skill optimization (one-off)
python gepa_optimize.py --skill .claude/skills/nextjs.md \
  --evaluator evaluators/nextjs_eval.py \
  --model claude-haiku-4-5-20251001 \
  --iterations 20
```

### Model selection guide

| Task | Model | Reason |
|---|---|---|
| Architecture decisions, spec review | Opus | Best reasoning, worth the cost |
| Code generation (subagents) | Sonnet | Good enough, 5× cheaper |
| GEPA evaluation runs | Haiku | Cheap, fast, good at pass/fail |
| Integration tests, lint | Haiku | Mechanical, no reasoning needed |

### The irreducible architect checklist

These are yours. Do not delegate them.

1. `spec.md` — define what and why
2. Review and approve interface contracts before agents start
3. Define evaluators for GEPA (what "correct" means)
4. Review skill diffs after GEPA iterations
5. Resolve blockers in `docs/blockers.md`
6. Set per-project token budget before starting

---

*Last updated: March 2026*  
*Stack: Claude Code native subagents + Docker Sandboxes + spec-kit + DSPy/GEPA + ccusage*
