# agent-first-repo

An [Agent Skill](https://code.claude.com/docs/en/skills) for setting up — or auditing — a
repository so coding agents can work in it effectively.

The premise: **agent effectiveness is determined by the environment, not the prompt.** When an
agent struggles, the fix is rarely a better prompt; it's a missing map, a missing invariant, or a
missing feedback loop in the repo. This skill encodes that into three concrete modes.

## What it does

| Mode | Trigger | Output |
|------|---------|--------|
| **Init** | "set up this repo for agents", "new project" | Docs skeleton, `CLAUDE.md`/`AGENTS.md` as a ~100-line map, `ARCHITECTURE.md` with dependency rules, mechanical enforcement, exec-plan template |
| **Audit** | "how agent-ready is this repo?" | 16-point scorecard across context, enforcement, workflow, and doc health, plus the three highest-leverage fixes |
| **Hygiene** | "sweep", "quality check" | Recurring pattern/doc-drift sweep and per-module quality grades |

The opinionated parts worth stealing even if you don't install it:

- **Entry-point file is a map, not an encyclopedia.** ~100 lines that point at deeper docs.
  The everything-file rots and crowds out the actual task.
- **Pick an enforcement level per constraint** — docs, lint rule, structural test, or CI gate.
  Documentation alone does not prevent drift.
- **Lint error messages are agent context.** Every custom rule states the remediation, because
  the failure text is what the agent reads next.
- **Anti-patterns beat positive rules.** An explicit "what NOT to do" list saves more agent time
  than the equivalent list of best practices.

## Install

The skill is a single self-contained `SKILL.md` at the repo root, so cloning the repo *as* the
skill directory works directly.

**Claude Code** (personal, all projects):

```sh
git clone https://github.com/rhlsthrm/agent-first-repo.git ~/.claude/skills/agent-first-repo
```

**Claude Code** (one project, committed for the team):

```sh
git clone https://github.com/rhlsthrm/agent-first-repo.git .claude/skills/agent-first-repo
```

**Oh My Pi / OMP:**

```sh
git clone https://github.com/rhlsthrm/agent-first-repo.git ~/.omp/agent/skills/agent-first-repo
```

**Any other harness:** the file is plain Markdown with YAML frontmatter (`name`, `description`).
Drop `SKILL.md` wherever your agent loads instructions from, or paste it into a prompt.

Then ask your agent: *"audit this repo for agent-readiness"* or *"set this repo up agent-first"*.

## Credit

Built on the "Harness Engineering" framing — humans steer, agents execute, and the environment
determines how well agents execute.

## License

MIT — see [LICENSE](LICENSE).
