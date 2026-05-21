# Exploration Map — Phase A

A map of the areas to be explored during brainstorming. Explore in approximate priority
order for the project type, but the map is a graph, not a sequence — return visits are
allowed when new insights surface.

The map describes *what* to explore. The technique for *how* is in
`brainstorming-techniques.md`.

---

## Tier 1 — always critical

### Problem & motivation
What does the product solve? Why does the problem exist? Who suffers from it today, and
how do they solve it now? Is the problem real or hypothetical?

### Target audience & use cases
Who is the user? A single person, multiple roles, organizations? What are the concrete
scenarios where the product is used? What is primary use, what is side use?

### Scope and boundaries
What is in scope? What is *deliberately* outside? Where is the boundary to other
systems or tools? Cutting scope is design work — note it actively.

### Core features
What is the product? Not everything that *could* exist, but what must exist for the
product to be meaningful. Rank if possible.

---

## Tier 2 — usually important

### User flow / interaction model
How does the user navigate? What are the main views, how are they connected? Is it CLI,
GUI, TUI, hybrid? What are the most frequent flows?

### Stack & technology choices
Which languages, frameworks, libraries? Motivation behind the choice (preference, AI
support in the ecosystem, hardware, distribution, existing competence). If a codebase
exists, stack can often be read directly from `go.mod`/`package.json`/`pyproject.toml`/
`Cargo.toml` without asking.

### Architecture
What components does the system consist of? How do they talk to each other? Where are
the boundaries? Monolith, modular monolith, microservices, client-server, standalone?

---

## Tier 3 — project-dependent

### Data model
Relevant if the product has persistent state. Which entities, which relationships,
which constraints? A schema sketch in text form is enough; full DDL can come later.

### Security model
Relevant if the product handles sensitive data, external users, or has attack surface.
What may happen, what may not? Auth model, encryption, logging, GDPR.

### Distribution
Relevant when the product is going to reach a user. CLI install, container, package
registry, binary from Releases, SaaS deployment? Often left open until later phases.

### Market & competition
Relevant for commercial products. Which competitors exist? What is the differentiation?
Who is the buyer, what do they pay today?

### Business model
Relevant for commercial products. How is money made? Subscription, license, freemium,
investor-driven? Exit strategy if relevant.

### Branding / tone
Relevant for products with user interfaces or marketing. Visual identity, voice, how
the product *feels*.

---

## Always (regardless of tier)

### Risks and assumptions
Which assumptions does the design rest on? Which risks exist? Which mitigations are
planned or possible?

### Success criteria
When is the product "done enough"? What is the minimum acceptable outcome? How is it
measured that the design worked?

---

## Heuristic for tier reshuffling per project type

The default order above suits most projects. Adjust per project type:

**Learning project** (the operator is building to learn)
- Stack and scope to tier 1 (the learning value is in the technology choice and
  scoping)
- Market, business model omitted
- Core features can be trimmed — over-ambition is the most common failure mode

**Commercial SaaS / product for external use**
- Target audience and scope to tier 1
- Market and business model to tier 2
- Security model and distribution to tier 2
- Branding to tier 2 if B2C or sales-dependent

**Internal tool** (the operator or a small group uses it themselves)
- Scope and integrations to tier 1
- Market omitted
- Business model omitted
- Branding often omitted

**Library / CLI for other developers**
- API design and scope to tier 1
- Distribution to tier 2
- User flow is replaced by "developer experience" — how is it discovered, installed,
  called?

**Agent system** (AI agent or agent cluster working autonomously)
- Security model to tier 1 (what the agent may do, what it may not)
- Data model to tier 1 (state, observations, memory)
- Distribution to tier 2

---

## Exit criterion

Phase A concludes when all tier 1 areas for the project type are covered, no unresolved
tier 2 areas of significance remain, the license has been explicitly decided (see
SKILL.md § Explicit decisions in Phase A), and the operator has nothing more to add or
change.

The agent signals with a short summary of what was covered and an explicit question:
"Anything you want to add before I generate the package?"
