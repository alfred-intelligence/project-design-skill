# Agent Instruction Patterns

Reusable patterns for 04-agent-instructions.md depending on project type.

> **Important:** 04-agent-instructions should *reference* `05-engineering-handbook.md`,
> `06-ci-cd-plan.md`, and `07-agent-loop.md` rather than duplicating rules. Those three
> documents are the agent's framework; 04 tells the agent to follow them. Duplication
> causes drift and contradictions.

---

## Pattern A: Single agent (small project)

Used when a project is driven by one AI agent without coordination needs.

Focus: clear init, sharp guardrails, concrete output format.

```markdown
## Init sequence

1. Read `01-whitepaper.md` to understand the project
2. Read `03-short-horizon.md` to understand the next step
3. Read `05-engineering-handbook.md` to understand the rules (commits, PR, releases)
4. Read `07-agent-loop.md` to understand the reporting structure
5. Verify the environment works: [command]
6. Begin Step 1; report according to 07
```

---

## Pattern B: Pipeline (sequential agents)

Used when Agent A feeds Agent B who feeds Agent C.

```markdown
## Data flow

Agent A (Collection) → `data/raw/` → Agent B (Processing) → `data/processed/` → Agent C (Output)

Each agent must:
- Verify that the input folder exists and is non-empty before starting
- Write a `status.json` in its output folder when done
- Never start the next step without the previous status.json being present
```

---

## Pattern C: Parallel agents with a shared codebase

Used when several agents work in the same repo but with separate areas of
responsibility.

```markdown
## File boundaries

| Agent | Owns | Reads (read-only) |
|-------|------|-------------------|
| Frontend agent | `src/ui/` | `src/api/types.ts` |
| Backend agent | `src/api/` | `src/ui/types.ts` |
| Test agent | `tests/` | `src/` (all) |

Rule: Never write to another agent's owned area.
```

---

## Pattern D: AI agent working against an existing repo

Used when the agent steps into an ongoing project.

```markdown
## Onboarding steps

1. Read `README.md`
2. Run `git log --oneline -20` to understand history
3. Run the tests: `[test command]` — all must be green before you start
4. Read `ARCHITECTURE.md` if it exists
5. Identify and list the 3–5 most important files for the task
```

---

## Guardrail library (pick relevant)

```markdown
**General:**
- Never commit directly to `main` or `master`
- Always run linting/tests before reporting a step as done
- Never change configuration files (`.env`, `docker-compose.yml`) without explicit
  permission
- Commit per logical change; never batch unrelated changes into one commit. In
  `spec-first` mode this is non-negotiable — squash-merge cannot be relied on, since
  release-please reads each commit since the last tag. In `iterative` mode the rule
  is still preferred, but PR-title Conventional Commits + squash-merge is acceptable.

**Spec-first mode (extra):**
- Narrate progress at each step. Write a short line stating which step is starting
  and what concrete action is about to happen; another line when the step finishes.
  Silent work during long autonomous runs makes the operator think the agent has
  stalled when it has not. Don't be silent.

**Filesystem:**
- Create a backup before modifying an existing file > 100 lines
- Never create files outside the project directory

**Communication:**
- Report blockers immediately — don't wait until the end
- Ask one question at a time, not several at once
- If you're unsure about scope: do the minimum, then ask

**Security:**
- Never log secrets, tokens, or passwords
- Never put credentials in code files
```

---

## Placeholder conventions in generated instructions

When 04-agent-instructions includes example actors (multi-agent patterns, contributor
examples, conflict scenarios), use the placeholder scheme from SKILL.md § Identity
neutrality:

- `operator` for the project's maintainer
- `alice` and `bob` for example third-party actors
- `acme-org` for an example organization
- `operator@example.com`, `alice@example.com` for example emails
- `example.com` for example domains

Substitute real identifiers only after the Phase A → Phase B confidentiality checklist
has explicitly confirmed which identifiers may appear in committed artifacts.
