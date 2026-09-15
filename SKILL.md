---
name: agent-first-repo
description: Initialize or audit a repository for agent-first development. Sets up structured docs, ARCHITECTURE.md, AGENTS.md as a map, mechanical enforcement via linters/tests/hooks, and observability wiring. Use when starting a new project, onboarding agents to an existing repo, or auditing agent-readiness.
---

This skill guides setting up (or auditing) a repository so that coding agents can work effectively. Based on OpenAI's "Harness Engineering" principles: humans steer, agents execute, and the environment determines agent effectiveness.

The user triggers this when: starting a new project, adding agent support to an existing repo, or asking "how agent-ready is this repo?"

## Core Philosophy

Agent effectiveness is determined by the environment, not the prompt. When an agent struggles, the answer is never "try harder" — it's "what's missing from the environment?"

Three laws:
1. **If the agent can't see it, it doesn't exist.** Knowledge in Slack, Notion, or people's heads is invisible.
2. **Enforce invariants, not implementations.** Tell agents the boundaries; let them solve within them.
3. **Context is scarce.** A giant instruction file crowds out the task. Use progressive disclosure: the entry-point file holds facts that apply every session, while procedures become skills (`.claude/skills/<name>/SKILL.md`) and area-specific rules become path-scoped files (`.claude/rules/*.md` with `paths:` frontmatter). Both load only when relevant.

## Mode: Init (new repo or first-time setup)

### Step 1 — Assess the project

Before generating anything, understand:
- What language/framework? (determines linter and test tooling)
- Monorepo or single package?
- What's the domain? (determines architecture layers)
- Does an AGENTS.md or CLAUDE.md already exist?
- Is there existing CI?

### Step 2 — Create the docs skeleton

```
AGENTS.md              # ~100 lines max. The map. Points to deeper docs.
CLAUDE.md              # One line: `@AGENTS.md`. Claude Code reads this name, not AGENTS.md.
ARCHITECTURE.md        # Domain map, package layering, dependency directions
docs/
  design-docs/
    index.md           # Catalog of design decisions with status
  exec-plans/
    active/            # In-progress execution plans
    completed/         # Done plans (kept for agent reference)
  references/          # llms.txt files, API docs, dependency guides
  CONVENTIONS.md       # Naming, file structure, error handling patterns
  TESTING.md           # Test strategy, what to test, how to run
  SECURITY.md          # Auth patterns, input validation, secrets handling
```

Not every project needs every file. Start with what's relevant:
- **Always**: AGENTS.md, ARCHITECTURE.md
- **If multi-step features planned**: docs/exec-plans/
- **If 2+ engineers or agents**: docs/CONVENTIONS.md
- **If external APIs or complex deps**: docs/references/

### Step 3 — Write AGENTS.md as a map

The entry-point file should be ~100 lines. It contains:

1. **One-paragraph project description** — what this is, what it does
2. **Tech stack** — language, framework, key dependencies, package manager
3. **How to run** — dev server, tests, lint, build (or point to Makefile)
4. **Architecture pointer** — "See ARCHITECTURE.md for domain map and layer rules"
5. **Key conventions** — 3-5 most important rules (the ones agents violate most)
6. **Doc index** — where to find deeper docs, with one-line descriptions
7. **What NOT to do** — explicit anti-patterns (these save more agent time than positive rules)

Anti-pattern: Don't put the full architecture, all conventions, every API pattern, and the test strategy in this file. That's the encyclopedia approach — it rots and overwhelms.

Name it `AGENTS.md`. It is an open format read by Codex, Cursor, Copilot, Gemini CLI, Zed, Aider and others, and is used by 60k+ public repos. Claude Code is the exception: it reads `CLAUDE.md`, so add a `CLAUDE.md` whose first line is `@AGENTS.md` (or `ln -s AGENTS.md CLAUDE.md`) and put any Claude-specific instructions below the import. One source of truth, no duplication.

### Step 4 — Write ARCHITECTURE.md

This is the structural map. Include:

1. **High-level diagram** (Mermaid) showing domains/packages and their relationships
2. **Layer definitions** — what each layer does, what it may depend on
3. **Dependency direction rules** — explicit "A may import B, but B must not import A"
4. **Cross-cutting concerns** — how auth, logging, config, errors flow through layers
5. **Where to add new things** — "New API endpoints go in src/routes/, new business logic in src/services/"

### Step 5 — Mechanical enforcement

Documentation alone doesn't prevent drift. For each constraint that matters, pick the enforcement level:

| Level | Mechanism | When to use |
|-------|-----------|-------------|
| 1. Docs only | ARCHITECTURE.md, CONVENTIONS.md | Soft preferences, style guidance |
| 2. Lint rule | Custom ESLint/Biome/clippy rule | Naming, imports, file structure |
| 3. Pre-action hook | `PreToolUse` hook that exits non-zero | Protected paths, forbidden commands — blocks before the edit lands |
| 4. Structural test | Unit test that asserts architecture | Dependency directions, layer boundaries |
| 5. CI gate | Fails the build | Security invariants, critical correctness |

Levels 1 and 2 are context: the agent can read them and still decide otherwise. A hook is not — it runs deterministically and can refuse the action outright, which is what you want for anything an instruction file merely asks nicely about. It also returns its stderr to the agent, so the same remediation rule below applies.

For custom lint rules: **always include remediation instructions in the error message.** When a lint fails, the error message becomes agent context. "Error: service layer must not import from UI layer. Move this logic to src/services/ or create a shared type in src/types/." is 10x more useful than "Import violation."

Minimum enforcement for any project:
- [ ] Import/dependency direction rules (structural test or lint)
- [ ] File size limits (lint — prevents agent sprawl, suggest 300-500 lines)
- [ ] Naming conventions (lint)
- [ ] Test coverage requirements (CI gate — at least for new code)

### Step 6 — Agent observability (project-specific)

Wire up what's relevant to the project type. Whatever the type, record the launch recipe rather than leaving it to inference: `/run-skill-generator` gets the app running from a clean environment and commits what worked to `.claude/skills/run-<name>/`, so `/run`, `/verify` and every other agent in the repo follow it instead of rediscovering the build.

**For web/frontend projects:**
- Make the app bootable per git worktree (isolated instances)
- Wire screenshot/DOM snapshot capability for agent validation
- Add visual regression tests agents can run

**For backend/API projects:**
- Structured logging queryable in dev (not just console.log)
- Health check endpoints agents can hit to verify their changes work
- Request/response examples in docs/references/ for key APIs

**For infrastructure/DevOps:**
- Dry-run modes for destructive operations
- Plan/apply separation (like Terraform) so agents can preview changes
- Rollback instructions in docs/

### Step 7 — Execution plan template

Create `docs/exec-plans/TEMPLATE.md`:

```markdown
# Plan: [Title]

## Status: active | completed | abandoned
## Started: YYYY-MM-DD
## Owner: [who is driving this]

## Goal
What we're building and why. 2-3 sentences.

## Approach
How we're building it. Key decisions and trade-offs.

## Tasks
- [ ] Task 1 — description
- [ ] Task 2 — description

## Decision Log
- **YYYY-MM-DD**: Decision made — rationale

## Open Questions
- Question that needs answering before proceeding
```

## Mode: Audit (existing repo)

When auditing an existing repo for agent-readiness, check each item and report:

### Checklist

**Context & Navigation**
- [ ] Entry-point file exists (AGENTS.md, with CLAUDE.md importing it) and is under 200 lines
- [ ] ARCHITECTURE.md exists with domain map and dependency rules
- [ ] Docs directory exists with indexed design decisions
- [ ] Key decisions are in-repo (not only in Slack/Notion/Google Docs)

**Mechanical Enforcement**
- [ ] Dependency directions enforced via lint or structural test
- [ ] File size limits enforced
- [ ] Naming conventions enforced
- [ ] Custom lint error messages include remediation instructions
- [ ] CI runs all enforcement on every PR

**Agent Workflow**
- [ ] App is runnable from a clean checkout (documented in 3 steps or fewer)
- [ ] Tests can run in isolation (no shared state, no manual setup)
- [ ] Dev server bootable per worktree (if applicable)
- [ ] Structured logging available in dev

**Documentation Health**
- [ ] No stale docs (architecture matches actual code)
- [ ] Anti-patterns documented (what NOT to do)
- [ ] Common error remediation documented

### Output

Present results as a scorecard:
```
Agent-Readiness Audit: [repo-name]
Context & Navigation:     [X/4]
Mechanical Enforcement:   [X/5]
Agent Workflow:            [X/4]
Documentation Health:      [X/3]
Overall:                   [X/16]

Top 3 improvements (highest leverage):
1. ...
2. ...
3. ...
```

## Ongoing: Code Hygiene Loop

After initial setup, maintain quality with a recurring process:

1. **Weekly sweep** — agent-driven scan for pattern deviations, stale docs, growing files
2. **Quality scoring** — grade each domain/module (A-D) in docs/QUALITY_SCORE.md, track over time
3. **Doc gardening** — periodically verify docs match reality, update or delete stale content
4. **Golden principles** — when you encode a new taste/style rule, add it to CONVENTIONS.md AND create a lint rule if mechanically enforceable

## What This Skill Does NOT Cover

- CI/CD pipeline setup (pipeline syntax is platform-specific)
- Specific framework scaffolding (use framework-specific tools)
- Git workflow / branching strategy (project-specific)
- Team process / PR review norms (org-specific)
