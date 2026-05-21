# Agent Loop Templates

Standard templates for `07-agent-loop.md` per strictness level. The skill should
*propose* a template based on 01–06 and ask the operator to confirm — not ask openly.

---

## Implementation mode interaction

The implementation mode (see SKILL.md § Explicit decisions in Phase A) is orthogonal
to strictness level but affects how the loop is written:

| Aspect | `iterative` | `spec-first` |
|---|---|---|
| Commit granularity in loop | Recommended | Mandatory (release-please) |
| Merge strategy | Squash (default) | Rebase (preserves commits for release-please) |
| Narration requirement | None | Implementer narrates each step start and end |
| Phase C frequency | Often, per milestone | Rare, only at scope shifts |
| Operator presence | Close, reviews as work unfolds | Can run autonomously (overnight, push-notification-driven) |

Template selection (A/B/C/D below) is by strictness level. The mode adjusts
*content* within the chosen template — the loop cadence and reporting cycle stay the
same, but commit/merge rules and narration come from the mode.

---

## The proposal model

The skill builds the proposal by looking at:

1. **Number of contributors** → `solo` / `solo+contrib` / `team`
2. **Project size** (small/medium/large)
3. **Risk level** (private side projects vs production systems with users)
4. **Release cadence** (continuous → stricter loop)

The result is presented as:

> "Based on the project, I propose the following agent loop:
> - Cadence: [per step / per branch / per step + daily]
> - Report channel: [chat / external notifier / status.json / PR description + daily
>   report]
> - Termination receipt: [operator's 'done' / merged PR / DoD + reviews]
>
> Approve, or adjust?"

---

## Template A: solo (small, private or otherwise)

### 07-agent-loop.md

````markdown
# [Project] — Agent loop

## Strictness level

**Chosen level:** `solo`

**Motivation:** Only the operator + AI. Quick feedback matters more than formal report
structure.

## Loop structure

**Cadence:** Per step (granular).

**Cycle:**
```
[init] → [execute 1 step] → [chat report] → [operator approves/adjusts] → [next step]
```

## Report format

**Format:** Markdown blob in chat.

**Template per step:**
```markdown
### Step X: [name] — [✅ done / ⚠️ blocker / ❌ error]

**What was done:** [brief]
**Output:** [files/results]
**Verified with:** [test/manual check]
**Next:** Step X+1 — [name]
```

## Reporting channel

**Default:** Directly in chat with the operator.
**For autonomous runs:** an external notification channel (Telegram, email, Slack,
etc.) — chosen by the operator based on what they monitor most often.

## Frequency and triggers

- At the end of steps
- At a blocker (immediately)
- Not per commit (too chatty)

## Escalation rules

| Situation | Action |
|-----------|--------|
| Blocker | Report directly, wait for operator's response |
| Two failed attempts | Pause, escalate briefly |
| Unclear requirement | Ask specifically, don't continue by guessing |
| Critical file missing | Report which one and why |

## Operator's follow-up rhythm

- Read each step report
- Confirm or adjust before the next step
- Default reply `ok` unless otherwise stated

## Termination criteria

- [ ] All steps in `03-short-horizon.md` done
- [ ] Operator acknowledges "done"

## Error handling at loop level

| Scenario | Handling |
|----------|----------|
| Agent gets stuck | Pause, report, wait |
| Operator absent | Finish current step, document, stop |
| Conflict between documents | Escalate, ask which applies |
| External dependency down | Report, retry after 5 min, escalate after 3 attempts |
````

---

## Template B: solo+contrib (open-source solo or shared private)

### 07-agent-loop.md

````markdown
# [Project] — Agent loop

## Strictness level

**Chosen level:** `solo+contrib`

**Motivation:** The operator is the primary developer but the repo accepts external
PRs. The loop structure needs to leave traceable artifacts (commits, PR descriptions)
behind so other contributors can follow along.

## Loop structure

**Cadence:** Per branch/PR.

**Cycle:**
```
[init] → [create branch] → [execute N steps on branch] → [open PR with report]
       → [CI runs] → [operator self-reviews] → [merge] → [next branch]
```

## Report format

**Per commit:** Conventional Commits messages.

**Per PR:** PR description following `bootstrap/.github/PULL_REQUEST_TEMPLATE.md`.

**Per step (internal, optional):** `status.json` in the `reports/` folder, committed
to the branch:

```json
{
  "step_id": "string",
  "status": "in_progress|done|blocked|failed",
  "started_at": "ISO8601",
  "completed_at": "ISO8601 | null",
  "summary": "string",
  "artifacts": ["path/to/file"],
  "blockers": ["string"],
  "next_step": "string | null"
}
```

## Reporting channel

- **Per commit:** git log
- **Per PR:** GitHub PR + status checks
- **Work in progress:** branch + optional WIP PR with `draft` flag
- **Urgent blockers:** chat with the operator

## Frequency and triggers

- Commit per logical change (Conventional Commits)
- Push per work day or completed logical unit
- PR when feature work is done
- Blocker report immediately in chat

## Escalation rules

| Situation | Action |
|-----------|--------|
| Blocker | Pause the branch, report in chat |
| CI red repeatedly | Report, don't guess |
| Unclear review feedback | Ask specifically in the PR thread |
| Conflict with `main` | Rebase, resolve, continue; escalate if conflict is large |

## Operator's follow-up rhythm

- PRs are reviewed at creation and at update
- Self-review within 1 work day
- CI green + self-review approved → merge
- External PRs reviewed within 2 work days (SLA in CONTRIBUTING.md)

## Termination criteria

- [ ] PR merged to `main`
- [ ] CI green after merge
- [ ] No regressions in staging (if applicable)
- [ ] Linked issue closed

## Error handling at loop level

| Scenario | Handling |
|----------|----------|
| Agent stuck on a branch | Mark PR `draft`, report, wait |
| Operator absent | Complete unstarted steps, push to branch, stop without merging |
| Conflict between documents | Report in PR description, ask the operator |
| External CI down | Wait and retry, escalate after 1 hour |
````

---

## Template C: team (several contributors)

### 07-agent-loop.md

````markdown
# [Project] — Agent loop

## Strictness level

**Chosen level:** `team`

**Motivation:** Several regular contributors. Formal reporting is necessary for
synchronization. The AI agent is treated as a contributor with the same requirements as
human developers.

## Loop structure

**Cadence:** Per step + daily summary.

**Cycle:**
```
[init] → [execute 1 step] → [status.json + commit] → [verification]
       → [next step or branch end] → [PR when feature done]
       → [code review (peer)] → [merge]

Daily: 16:00 → daily report to reports/daily/YYYY-MM-DD.md
```

## Report format

**Per step:** `status.json` in the `reports/` folder, committed directly:

```json
{
  "step_id": "string",
  "status": "in_progress|done|blocked|failed",
  "started_at": "ISO8601",
  "completed_at": "ISO8601 | null",
  "agent_id": "string",
  "branch": "string",
  "summary": "string",
  "artifacts": ["path/to/file"],
  "blockers": ["string"],
  "next_step": "string | null"
}
```

**Per day:** Markdown report to `reports/daily/YYYY-MM-DD.md`:

```markdown
# Daily report YYYY-MM-DD

## Done today

- [Step X]: brief description + commit sha
- [Step Y]: ...

## Blockers

- [Blocker 1]: affects [step], waiting on [person/info]

## Tomorrow

- [Step Z]: planned
- [Step W]: planned

## Risks

- [Risk]: mitigation in progress
```

**Per PR:** Following `bootstrap/.github/PULL_REQUEST_TEMPLATE.md`, including test plan
and rollback plan.

## Reporting channel

- **Per step:** `status.json` in repo (committed)
- **Per day:** `reports/daily/` folder (committed)
- **Per PR:** GitHub PR + required status checks
- **Urgent blockers:** issue with label `urgent` + Slack/chat
- **Postmortem:** `docs/postmortems/YYYY-MM-DD-<incident>.md` for major errors

## Frequency and triggers

- Status update at each step's start and end
- Daily report at 16:00
- PR when feature complete
- Blocker report immediately
- Weekly summary on Friday (aggregated from daily reports)

## Escalation rules

| Situation | Action |
|-----------|--------|
| Blocker | Issue with label `urgent`, ping the responsible CODEOWNER |
| Two failed attempts | Pause, escalate to team lead |
| Unclear requirement | RFC issue if larger, direct question if smaller |
| Conflict between documents | Escalate to maintainer |
| Security incident | Follow SECURITY.md, NOT GitHub Issues |

## Operators' follow-up rhythm

- Maintainer reads daily reports daily
- Reviewers review PRs within 2 work days (SLA)
- Weekly meeting: go through blockers and risks
- Monthly meeting: evaluate the loop, adjust as needed

## Termination criteria

- [ ] All steps in `03-short-horizon.md` marked done
- [ ] All milestones in phase X met per definition of done
- [ ] CI green in `main` after the final merge
- [ ] Postmortem written if major incidents occurred
- [ ] Maintainer acknowledges "done" in the weekly meeting

## Error handling at loop level

| Scenario | Handling |
|----------|----------|
| Agent gets stuck | Mark PR draft, status: blocked, escalate to team lead |
| Operator absent | Continue per plan if possible; pause at a blocker |
| Conflict between documents | Maintainer decides; update the document |
| External dependency down | Log incident, retry, fall over to backup if available |
| Regression detected | Stop the loop, rollback per 06, postmortem |
````

---

## Template D: solo (learning-optimized)

A variant of Template A for learning projects — where the operator is building
primarily to learn the technology or domain, not to ship as fast as possible. Loop
cadence is deliberately sparser than standard `solo` so the operator writes a lot of
code themselves between agent interactions. The termination receipt includes a
"do I understand the code?" confirmation that is non-negotiable.

### 07-agent-loop.md

````markdown
# [Project] — Agent loop

## Strictness level

**Chosen level:** `solo` (learning-optimized variant of Template A)

**Motivation:** Only the operator + AI. Learning project → loop cadence is deliberately
sparser than standard `solo` so the operator has time to write code themselves between
agent interactions. Quick feedback matters more than formal report structure, but the
termination receipt includes an explicit "do I understand the code?" confirmation that
isn't in the standard template.

## Loop structure

**Cadence:** Per milestone (M1–MN per `02-long-horizon.md`), not per step.

**Cycle:**
```
[init: read 01–07, set up the environment]
   ↓
[execute N steps from 03] ← (step report only at a blocker or for questions)
   ↓
[milestone reached]
   ↓
[milestone report (chat) + termination receipt]
   ↓
[operator acknowledges: "I understand the code", CI green, commits OK]
   ↓
[next milestone]
```

**Why not per step?** A per-step loop becomes transactional — the operator approves the
agent's output and barely touches the code. For a learning project that defeats the
purpose. Per milestone gives the operator room to write, change, walk away, and come
back.

## Pedagogical mode (core point)

This is what separates the learning loop from standard solo.

**On a new concept in the step:**
1. Identify the concepts that come up
2. Explain each new concept in 3–8 sentences — what it is, why it exists, how it
   differs from the operator's home language if relevant
3. Then produce code, with inline comments pointing at the concepts
4. After the code: mention common mistakes one could have made here

**On compile errors:** show the full error message, explain what the compiler *means*,
propose at least two solutions, let the operator choose.

**On logic errors/panics:** root cause before patch.

**Pedagogical mode is NOT:**
- Explaining basic syntax in every file
- Repeating the same explanation every time the same concept appears
- Being condescending — the tone is collegial, not lecturing

## Report format

### Milestone report

```markdown
## Milestone Mx — [name]

### Status: ✅ done / ⚠️ partial / ❌ blocked

### What was done
- Step X.1: [brief]
- Step X.2: [brief]

### Termination receipt
- [ ] All steps in the short horizon for the phase marked done
- [ ] Build succeeds
- [ ] Tests pass
- [ ] Linter clean
- [ ] CI green on the latest push in `main`
- [ ] **Operator has confirmed: "I understand what each file in src/ does"**

### New concepts introduced in this milestone
- [concept 1]: [what the operator now understands about it]

### Remaining for the next milestone
- [TODO 1]

### Design choices made along the way (if any)
- [choice 1]: [motivation, optional link to ADR]

### Next: M(x+1) — [name]
```

### At a step (informal)

No formal format. A chat message is enough. At a blocker: concrete problem + error
message + at least two proposed solutions.

## Reporting channel

- **Default:** chat with the operator
- **Milestone report:** markdown comment in the milestone PR (one PR per milestone
  closes the loop cycle)
- **Urgent blockers:** chat immediately
- No other channels — no status.json, no `reports/` folder

## Termination criteria

### Per milestone

All boxes in the termination receipt above must be checked. The **non-negotiable** one
is "Operator has confirmed: 'I understand what each file in src/ does'". If the answer
is "well, X.rs is still unclear" — the milestone isn't done until it is, either through
refactoring, documentation, or the operator's own work with the code.

### For the whole project

- All milestones closed per their termination receipts
- v0.1.0 (or equivalent) published
- Distribution exists in at least one form
- README gives an outsider enough to install and run

## Anti-pattern: what the loop *should not* do

- **Not per-commit reporting** — chatty and meaningless
- **Not autonomous runs** — learning project, not automation. The agent always waits
- **Not fast-generating** — if a milestone is done in 2 days instead of 2 weeks,
  something is wrong: scope underestimated or the agent did too much for the operator
- **Not heavy report structure** — if the milestone report generates pro forma text
  that just gets skimmed, simplify the template
````

---

## Proposal algorithm (for the skill to follow)

When the skill silently assesses the loop in Phase A and presents in Phase B:

```
level = assess_level(
  contributors = brainstorming_signals,
  public = brainstorming_signals,
)

size = assess_size(
  phase_count = from_02,
  step_count = from_03,
  agent_count = from_04,
)

risk_level = assess_risk(
  production_system = from_01,
  external_users = from_01,
  release_cadence = brainstorming_signals,
)

learning = assess_learning(
  stated_learning_value = from_01,
  new_stack_for_operator = from_01,
)

template = select_template(level, size, risk_level, learning)

present(template) + ask_for_approval
```

**Combination rules:**
- `solo` + learning = true → use **Template D** instead of Template A
- `team` + size `small` → use Template C but trim daily report to a weekly report
- `solo` + risk_level `high` (e.g. solo running a production SaaS) → use Template B
  instead of A
- `solo+contrib` + size `large` → consider lifting in Template C elements
  (status.json per step)
- `solo+contrib` + learning = true → still Template B, but include pedagogical-mode
  elements from Template D

---

## Update mode — assessment of the current loop

When the skill is run against an existing design package:

1. Read the current `07-agent-loop.md`
2. Identify which template it corresponds to (A/B/C)
3. Compare against the current reality:
   - Has the number of contributors changed?
   - Has the project moved public/private?
   - Has the release cadence changed?
   - Has the size grown/shrunk?
4. **Leave a recommendation both upward and downward:**
   - "The loop is on `solo` but you have now opened the repo publicly and gained two
     recurring contributors. Recommend `solo+contrib` (Template B). Affects:
     branch-protection.json, CONTRIBUTING.md, 07-agent-loop.md."
   - "The loop is on `team` but the team has shrunk to just you. Recommend trimming to
     `solo+contrib` (Template B) — a daily report is overkill, status.json is enough.
     Affects: 07-agent-loop.md."
5. Wait for approval before changes.
