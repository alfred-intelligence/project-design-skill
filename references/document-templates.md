# Document Templates

Detailed templates for each document in the project-design skill.

---

## Template: Whitepaper (01-whitepaper.md)

```markdown
# [Product name] — Whitepaper

> [A single sentence describing what the product does and for whom.]

## Problem

[What problem is being solved? Why does it exist? Who is affected?]

## Solution

[How does the product solve the problem? Describe at a conceptual level.]

## Architecture

[Text-based architecture sketch or component list]

Example:
- **Frontend**: React SPA, communicates with a REST API
- **Backend**: FastAPI, Python 3.12
- **Database**: PostgreSQL (persistent) + Redis (cache)
- **Auth**: JWT via OAuth2

## Design choices and motivation

| Choice | Alternatives considered | Motivation |
|--------|-------------------------|------------|
| PostgreSQL | MongoDB, SQLite | Relational data, ACID requirement |
| FastAPI | Flask, Django | Typed contracts, fast iteration |

## Dependencies

- [List external services, APIs, libraries of significance]

## Limitations and risks

- [Risk 1]: [Mitigation]
- [Risk 2]: [Mitigation]

## Future direction

[What is outside scope for now but may come later?]

## Assumptions for Phase B

Operationalization assumptions taken during Phase A, for confirmation at Phase B start.
Preserved here so they are not lost if the session changes.

(Note: license is **not** an assumption — it was decided explicitly in Phase A and is
recorded as a fact, not a recommendation.)

- **Strictness level:** [`solo` / `solo+contrib` / `team`] (motivation: ...)
- **Repo:** [private / public] (motivation: ...)
- **Release cadence:** [continuous via release-please / calendar / ad-hoc] (motivation: ...)
- **Distribution:** [concrete choice or TBD] (motivation: ...)
- **CoC/SECURITY:** [generated / skipped] (motivation: ...)
- **Documentation license:** [CC-BY 4.0 / CC-BY-SA 4.0 / "All Rights Reserved"] (motivation: ...)
```

---

## Template: Long horizon (02-long-horizon.md)

```markdown
# [Project name] — Long horizon

## Overview

| Phase | Name | Content | Estimate |
|-------|------|---------|----------|
| 1 | Setup & MVP | [short description] | 1–2 weeks |
| 2 | Core features | [short description] | 2–4 weeks |
| 3 | Beta | [short description] | 1–2 weeks |
| 4 | Launch | [short description] | 1 week |

## Phase 1: [Name]

**Goal:** [What should be done?]
**Done when:** [Concrete definition of done]
**Dependencies:** [What must be in place before this phase?]

[Repeat per phase]

## Milestones

- [ ] [Date/week]: [Milestone]
- [ ] [Date/week]: [Milestone]

## Risk buffer

[Which phases are sensitive to delays, and why?]
```

---

## Template: Short horizon (03-short-horizon.md)

> **Note:** Created in Phase A but *may be revised at Phase B start* because more
> information is available then (operationalization decisions are made, the repo
> exists, possible stack adjustments have been made). List concrete revisions and wait
> for approval before writing them.

````markdown
# [Project name] — Short horizon

Detail plan for [Phase X / Sprint 1 / "the next 2 weeks"].

## Step 1: Environment setup

**Task:** Set up development environment
**Output:** The project runs locally
**Commands:**
```bash
git clone ...
cd project
cp .env.example .env
docker compose up -d
```
**Done when:** `GET /health` returns 200

## Step 2: [Name]

**Task:** [What to do]
**Output:** [Concrete deliverable]
**Blocker:** [Anything that could stop this?]
**Done when:** [Verifiable criterion]

[Continue per step...]

## Critical path

Step 1 → Step 3 → Step 5 (these cannot be parallelized)

## Test points

- After step 2: [What to test?]
- After step 4: [What to test?]
````

---

## Template: AI agent instructions (04-agent-instructions.md)

> **Note:** Created in Phase A but *may be revised at Phase B start*. Once 05/06/07
> are in place, 04 can reference them concretely instead of duplicating rules. List
> revisions and wait for approval.

```markdown
# [Project name] — AI agent instructions

## Project context

[2–4 sentences about what is being built, in which stack, and what the goal is right
now. The agent should be able to understand the whole task without external context.]

**Rules and framing:** See `05-engineering-handbook.md`.
**CI/CD flows:** See `06-ci-cd-plan.md`.
**Loop, reporting, follow-up:** See `07-agent-loop.md`.

## Roles

### [Role 1: e.g. "Implementation agent"]

**Responsibility:** [What does this agent do?]

**Tools and permissions:**
- Read/write: `src/`, `tests/`
- Run: `pytest`, `npm test`, `docker compose`
- MUST NOT: commit to `main`, deploy, change `.env`

**Guardrails:**
- Ask the operator if a requirement is unclear instead of guessing
- Never overwrite files without reading them first
- Follow the commit conventions in 05 § Commits
- Follow the PR rules in 05 § PR process
- [Other constraints]

**Spec-first mode additions** (include only if `05-engineering-handbook.md` records
implementation mode = `spec-first`):

- **Commit granularity is mandatory.** One logical change per commit. No batching.
  release-please reads these commits to build the changelog; batched commits produce
  empty entries and missed version bumps. See `05 § Commits`.
- **Narrate progress.** At the start of each pre-approved step, write a short line
  in chat (or the configured reporting channel) stating which step is starting and
  what concrete action is about to happen. At the end, write a short line stating
  what was done. This prevents the operator from assuming the run has stalled during
  long autonomous sessions. Silent work is not acceptable in `spec-first` mode.

**Init steps:**
1. Read `01-whitepaper.md` (understand what is being built)
2. Read `03-short-horizon.md` (understand the next step)
3. Read `05-engineering-handbook.md` (understand the rules)
4. Read `07-agent-loop.md` (understand the reporting structure)
5. Run `[setup command]` to verify the environment
6. Begin Step 1 in the short horizon
7. Report per 07 after each completed step

---

### [Role 2: e.g. "Review agent"] (if applicable)

[Same structure]

---

## Communication protocol (if multiple agents)

See `references/agent-instruction-patterns.md` for standard patterns (Pipeline,
Parallel with shared codebase, Onboarding against an existing repo).

## Error handling

- If a step fails: log the error, retry max 2 times, escalate per 07 § Escalation.
- If a requirement is unclear: ask a specific question, wait for a response.
- If a file is missing: report which one and why it is needed.

## Output format

[What is the agent expected to deliver? Code files? Report? Test results?
Format for status reports: see 07.]
```

---

## Template: Engineering Handbook (05-engineering-handbook.md)

```markdown
# [Project name] — Engineering Handbook

## Implementation mode

**Chosen mode:** [`iterative` / `spec-first`]

**Motivation:**
[Why this mode? See SKILL.md § Explicit decisions in Phase A for the full description.]

Mode is referenced explicitly in Commit conventions, PR process, and Release process
below because it changes the rules in those sections.

## License

**Chosen license:** [E.g. MIT / Apache-2.0 / MPL-2.0 / AGPL-3.0 / "All Rights Reserved" (private)]

**Motivation:**
[Why this one? Permissive vs copyleft? Patent grant? Compatibility with dependencies?]

LICENSE file: see `bootstrap/LICENSE`.

## Documentation license

**Chosen license:** [E.g. CC-BY 4.0 / CC-BY-SA 4.0 / "All Rights Reserved"]

**Motivation:**
[Default mapping from source license (see
`references/engineering-handbook-templates.md` § Documentation license) or operator
override.]

**Scope:** Covers `README.md`, `docs/`, `{project}-design/`, and other top-level
`.md` files. Does not cover `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`,
or inline source-code comments.

LICENSE-docs file: see `bootstrap/LICENSE-docs`.

## Repo structure

```
[Project top level with short descriptions]
project/
├── src/              Source code
├── tests/            Tests
├── docs/             Documentation
├── .github/          GitHub config
└── ...
```

## Branch strategy

**Model:** [Trunk-based / GitFlow / GitHub Flow]

**Rules:**
- `main` is always deployable.
- Feature branches are named `<prefix>/<scope>-<description>`, e.g. `feat/auth-jwt`.
- Branches are merged via PR; no direct push to `main`.

## Commit conventions

**Standard:** Conventional Commits (`<type>(<scope>): <subject>`).

**Allowed types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `ci`,
`build`.

**Rules:**
- Subject in imperative form, ≤ 72 characters.
- Body explains *why*, not *what* (the *what* is visible in the diff).
- Breaking changes marked with `!` or `BREAKING CHANGE:` in the footer.
- One logical change per commit. In `iterative` mode this is a recommendation; in
  `spec-first` mode it is non-negotiable (release-please depends on it — see
  `references/engineering-handbook-templates.md` § Commit conventions).

Validation: [`commitlint` / manual / `gh` CLI hook].

## PR process

**PR size:** Preferably < 400 lines of diff. Larger PRs split or motivated in the
description.

**PR description should contain:**
- What changes (briefly)
- Why (link to issue or motivation)
- How it was tested
- Any migration or config changes

**Requirements for merge:**
- CI green (all workflows in 06).
- Required reviewer count: [`0` / `1` / `2`] (see branch-protection.json).
- No unresolved discussions.
- Conventional Commits-formatted title.

**Merge strategy:** [Squash / Rebase / Merge commit. Default rule of thumb: squash in `iterative` mode, rebase in `spec-first` mode.]
**Auto-merge:** [Enabled / Disabled — motivation]

## Code review routine

[What is reviewed? How quickly is review expected? Who owns which files (CODEOWNERS)?]

For `solo`: the AI agent does a pre-review against a checklist; the operator
self-reviews before merge.
For `solo+contrib`: at least 1 review (CODEOWNERS = the operator).
For `team`: 2 reviews + dismiss stale + required reviewers from CODEOWNERS.

## Documentation review

[Which docs-only PRs get a fast-track? Who owns ARCHITECTURE.md / README.md?]

## Release process

**Versioning:** SemVer (MAJOR.MINOR.PATCH).

**Release tooling:** [release-please / semantic-release / manual tag + GitHub Release]

**Release cadence:** [Continuous (per PR merge) / Calendar (biweekly) / Ad-hoc]

**Release steps:**
1. [Trigger]
2. [Generate changelog]
3. [Tag]
4. [Publish GitHub Release with notes]
5. [Optional deploy per 06]

## Maintenance

- **Dependency updates:** Dependabot (config: `bootstrap/.github/dependabot.yml`).
- **Security patching:** [SLA: 7 days for critical, 30 days for high.]
- **Deprecation:** [Process for marking and removing old APIs.]
- **EOL policy:** [When do old versions stop being supported?]
```

---

## Template: CI/CD plan (06-ci-cd-plan.md)

````markdown
# [Project name] — CI/CD plan

## Overview

| Workflow | Trigger | Purpose | Required for merge? |
|----------|---------|---------|---------------------|
| `ci.yml` | push, PR | Lint + test + build | Yes |
| `release.yml` | tag push | Build release artifacts, publish | No |
| `security.yml` | scheduled + push | CodeQL/security scan | [Yes/No] |
| ... | ... | ... | ... |

## Workflow: ci.yml

**Triggered on:** `push` to all branches, `pull_request` against `main`.

**Steps:**
1. Checkout
2. Setup [Go/Node/Python]
3. Cache handling
4. Lint ([golangci-lint / eslint / ruff])
5. Typecheck (if relevant)
6. Test (with coverage)
7. Build
8. Upload coverage as an artifact

**Exit policy:** Stop on first failure. Coverage thresholds: [e.g. 70%].

## Workflow: release.yml

**Triggered on:** push of tag `v*.*.*`.

**Steps:**
1. Checkout with full history
2. Setup
3. Build release artifacts (binaries / packages / container images)
4. Generate changelog
5. Create GitHub Release with assets
6. [Publish to package registry / Docker Hub / GHCR]

## Environments

| Environment | Purpose | Deploy trigger | URL |
|-------------|---------|----------------|-----|
| dev   | Daily development | Auto on push to `develop` | dev.example.com |
| staging | Pre-prod test | Auto on push to `main` | staging.example.com |
| prod  | Production | Manual approval on release tag | example.com |

## Deploy strategy

[Blue-green / Canary / Rolling / Direct]

## Rollback routine

1. [How quickly can rollback happen?]
2. [Who has the authority?]
3. [What triggers automatic rollback? (smoke test, error rate, etc.)]
4. [How is rollback communicated?]

## Observability

- **Logs:** [Where? Retention? Structured format?]
- **Metrics:** [Prometheus / Grafana / custom?]
- **Alerts:** [Thresholds for error rate, latency, etc.]
- **Tracing:** [OpenTelemetry / N/A]

## Self-hosted runners (if relevant)

[Specification: hostname, OS, hardware, access, security policy.]
````

---

## Template: Agent loop (07-agent-loop.md)

````markdown
# [Project name] — Agent loop

> Concretization of the execution loop — how the AI agent reports and how follow-up
> happens.

## Strictness level

**Chosen level:** [`solo` / `solo+contrib` / `team`]

**Motivation:** [Why this level? Solo = no external; team = formal reporting.]

## Loop structure

**Cadence:** [Per step / Per branch (PR-based) / Per phase / Per step + daily summary]

**Cycle:**
```
[init] → [execute N steps] → [report] → [operator review] → [approve/adjust] → [next]
```

[Adapt the arrows to the chosen cadence.]

## Report format

**Format:** [Chat blob (markdown) / status.json / PR description + commits / daily
report (markdown)]

**status.json schema (if relevant):**
```json
{
  "step_id": "string",
  "status": "in_progress|done|blocked|failed",
  "started_at": "ISO8601",
  "completed_at": "ISO8601 | null",
  "summary": "string",
  "artifacts": ["path/to/file1", "..."],
  "blockers": ["string", "..."],
  "next_step": "string | null"
}
```

## Reporting channel

[Where does the report land?]
- Chat with the operator (default at `solo`)
- An external notification channel (Telegram, email, Slack, etc.) for autonomous runs
- `reports/` folder in the repo (committed)
- PR description + commit messages (at `solo+contrib`)
- Daily report markdown to `reports/daily/YYYY-MM-DD.md` (at `team`)

## Frequency and triggers

[How often is reporting done?]
- At the start and end of steps
- At a blocker
- On a successful build/test
- [Daily summary at HH:MM]

## Escalation rules

| Situation | Action |
|-----------|--------|
| Blocker (cannot continue) | Report directly, wait for operator's response |
| Two failed attempts | Pause, escalate with brief summary |
| Unclear requirement | Ask a specific question, don't proceed by guessing |
| Critical file missing | Report which one and why, wait |
| Data-loss risk | STOP immediately, escalate |

## Operator's follow-up rhythm

[When is the operator expected to read the reports?]
- At each report (default `solo`)
- At each PR (default `solo+contrib`)
- Once per day (default `team`)

**Operator's responsibility:**
- Confirm approval of reports
- Resolve blockers within [SLA]
- Correct course as needed

## Termination criteria

**The loop ends when:**
- [ ] All steps in `03-short-horizon.md` are marked done
- [ ] All milestones in phase X are met per definition of done
- [ ] CI is green in `main`
- [ ] The operator acknowledges "done"
- [ ] [Any additional criteria]

## Error handling at loop level

| Scenario | Handling |
|----------|----------|
| Agent stuck | Pause, report, wait for input |
| Operator absent | Complete unstarted steps, document current state, stop without breaking anything |
| Conflict between documents 01–06 | Escalate, ask which document applies |
| External dependency down | Report, retry after [interval], escalate after [N attempts] |
````
