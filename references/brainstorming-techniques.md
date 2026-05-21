# Brainstorming Techniques — Phase A

The technique for *how* the exploration happens. *What* gets explored is in
`exploration-map.md`.

---

## Core rule: result before agreement

The goal of brainstorming is a good product on paper — not a satisfied operator in the
moment. Justified criticism serves the result. An operator who is corrected on a
substantive point but ends up with a better design is better served than one who is
mirrored into a mediocre outcome.

Mirroring behaviors (vocabulary mimicry, reflexive references to known products,
"as you said earlier" phrasing) are *not* techniques. They are appeasement and don't
belong in the method. If they emerge naturally in a concrete analysis, that's one
thing; as routine practice, they're wrong.

---

## Three input modes

Calibrated per area, often even within an area as new information surfaces.

### Too little info or uncertain
The operator has said something brief or vague and the area isn't covered. **Action:**
open question + a concrete example that shows what kind of answer is needed.

> Example: "When you say 'clean up junk in `$HOME`' — do you mean package caches and
> build artifacts (like `node_modules`, `target/`), or more like empty folders and
> duplicates? Both are reasonable starting points but lead to different scanners."

### Right amount of info
The operator has given enough for the area to be covered. **Action:** confirm briefly
and move on. No unnecessary follow-up questions.

> Example: "Noted: private repo for now, possibly open source later. Moving on to the
> next area."

### Lots of info with gaps
The operator has given a lot but something is missing, contradicts itself, or contains
a questionable choice. **Action:** point out the gap, propose a concrete addition or
change, motivate. Criticism always comes paired with a counter-proposal.

> Example: "You mention Temporal for background jobs. That's overkill for your actual
> need ('guaranteed delivery within a day during downtime') — Asynq or a pgqueue solves
> it at ten percent of the complexity. Temporal becomes relevant only when workflows
> are long-lived and coordinate multiple services. Want Asynq instead?"

---

## Question types per phase

Early in an area, when scope is unclear: **open-ended**.
> "What does a typical work session look like for you?"

When scope starts to land: **ranked or multi-select**.
> "Which of the following count as 'junk' you want to clean up? (tick multiple) —
> package caches / build artifacts / downloads / empty folders / top-N largest files"

For details: **specific questions, one at a time**.
> "Should scanning follow symlinks?"

For uncertainties or several valid paths: **A/B/C with tradeoffs**.
> "Three paths for the editor strategy: A — own editor (full control, more work), B —
> delegated to OnlyOffice (less work, less control), C — hybrid. Which way are you
> leaning?"

---

## Reframing on a new metaphor

When the operator gives a new frame ("imagine it's a workplace", "like Figma but for
X", "roughly like Obsidian"), reformulate the project in the new frame *immediately*.
Don't continue in the old one.

Reframing moments are often where the core is revealed. A product that starts as a
"note-taking app" can, via a metaphor, turn out to be a "case management system". That
changes the whole map.

---

## Summarize before the next question

Before each major transition: briefly list what's been heard, ask if it's correct, then
ask the next question. The operator has a chance to correct before the next round of
guesses starts.

> "Summary so far: a `$HOME` cleaner in Rust, hybrid category and drill-down view,
> dry-run default. Correct? Then we move on to the security model."

---

## Acknowledge knowledge gaps — ask, don't guess

When the agent doesn't have enough domain knowledge to proceed meaningfully: say so
directly. Better to ask than to assume. "I don't know enough about how legal work works
in practice to write this section — can you describe a typical case flow?" is always
better than an assumption that has to be rolled back.

---

## Identify what *not* to build

Cutting scope is design work. At each major decision: note what was *excluded* and why.
That list is as important as the feature list, and helps in Phase B when
operationalization needs to be prioritized.

---

## Spontaneous thoughts after a decision

When a section has landed: come unprompted with risks, opportunities, or blind spots
you see. That's where brainstorming differentiates itself from an interview — the agent
has its own agency and its own observations, not just follow-up questions.

> "Spontaneous thought: the boundary against the incumbent (you mentioned not wanting
> to compete with their invoicing module) is your strongest selling point so far, but
> it also means your own product must never start sneaking in invoicing features 'as a
> convenience'. That's a discipline you'll have to hold over time."

---

## Fact-checking on business and market decisions

Market assumptions, partnerships, regulatory requirements, ecosystem changes — search
before recommending, not after. External reality may have shifted since the operator's
last scan. A recommendation without factual grounding is speculation.

---

## Anti-patterns

Behaviors that look like good technique but aren't. Avoid actively.

### Mirroring instead of analysis
Sticking to the operator's vocabulary or comparing to known products ("like Figma",
"Notion-like") isn't a method in itself. If the comparison isn't motivated by the
analysis, don't make it. Words should serve understanding, not signal listening.

### Over-interpretation of priority answers
"Not a priority" is not "remove it". "I don't care much" is not "skip it". "Later" is
not "never". Confirm with a follow-up if the operator's response is negatively charged
but not explicitly rejecting.

### Stacking of assumptions
Presenting an interwoven conclusion that rests on three unconfirmed steps in a row.
Break it down. Confirm step by step. A single error in the middle topples the whole
conclusion.

### Backing off because the operator is irritated
Irritation is a signal, not an order. If the questions aren't producing value — back
off, ask simpler. If the operator is just tired or frustrated but the questions are
right — hold the line. The difference lies in whether the questions are *doing* work or
just generating friction.

### False consensus
Reading silence or short replies ("ok", "yes", "go") as full approval of a detailed
recommendation. For longer or complex proposals: ask for explicit confirmation of the
main points.

### Pseudo-criticism
Criticism that doesn't lead anywhere ("this is ambitious" without a counter-proposal,
"worth thinking about" without direction). If criticism can't be paired with a concrete
counter-proposal and a motivation — hold it until it can.

### Treating conversation context as disclosure consent
The operator may have mentioned identifiers (real handles, real org names, real
projects) earlier in the session or in memory. That is conversational context, not
permission to commit those identifiers into public artifacts. See SKILL.md §
"Phase A → Phase B confidentiality checklist". When in doubt about whether something
may go into a committed document, ask.
