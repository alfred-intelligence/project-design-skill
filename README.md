# project-design

A Claude Skill that assembles a complete planning package for software projects —
product design in Phase A, operationalization plus a GitHub bootstrap folder in
Phase B, and adaptive revisions in Phase C.

Works for CLI tools, web and SaaS apps, AI agent systems, libraries, and similar
projects. Triggers on English and Swedish phrasing equally.

## What it produces

**Phase A — Product design** (delivered to `{project}-design/`):

- `00-index.md` — overview
- `01-whitepaper.md` — product description + assumptions for Phase B
- `02-long-horizon.md` — coarse plan (phases, milestones)
- `03-short-horizon.md` — detail plan, nearest phase
- `04-agent-instructions.md` — AI agent instructions

**Phase B — Operationalization** (after the operator creates the repo and works
through a confidentiality checklist):

- `05-engineering-handbook.md` — license, branch, commits, PR, release, maintenance
- `06-ci-cd-plan.md` — workflows, environments, deploy, rollback, observability
- `07-agent-loop.md` — loop cadence, reporting format, escalation, termination
- `bootstrap/` — importable repo artifacts (LICENSE, README skeleton, `.gitignore`,
  `.editorconfig`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, GitHub
  workflows, issue/PR templates, labels, branch-protection JSON, dependabot config,
  stack-specific files)

**Phase C — Iteration**: detects an existing `{project}-design/` and proposes scope-
delta-driven revisions, with recommendations both upward and downward in strictness
level.

## How it differs from a linear interview

Brainstorming in Phase A is adaptive mapping (see `references/exploration-map.md`),
not a checklist. Most operationalization questions are assessed silently from the
brainstorming output and presented as recommendations at the Phase B handoff, with
three explicit exceptions:

- **Implementation mode** — `iterative` (design co-evolves with code, squash-merge,
  Phase C revisions are normal) or `spec-first` (design locked, granular commits,
  rebase-merge for release-please, implementer narrates progress for autonomous
  runs). Decided in Phase A *before* 03 and 04 are written, since the mode shapes
  their detail level.
- **License** — always decided explicitly in Phase A. Has legal and ecosystem
  consequences the operator must own.
- **Repo namespace, visibility, and any identifier disclosure** — clarified
  explicitly in the Phase A → Phase B confidentiality checklist before anything is
  committed to a repository.

The skill maintains a running list of open questions and audits them before any
major batch generation, so the operator doesn't have to backtrack through the
conversation to verify that nothing was missed.

## Identity neutrality

All generated documents default to identity-neutral placeholders (`operator`,
`alice`, `bob`, `acme-org`, `example.com`). Real identifiers appear in committed
artifacts only after explicit confirmation in the confidentiality checklist —
mention in conversation or memory is not consent to publish.

## Platform assumption

GitHub. All repo artifacts are generated in GitHub format (Actions YAML, `gh`-
compatible JSON). Deliberate choice for simplicity; porting to GitLab or similar is
left to the operator if needed.

## Installation

This is a Claude Skill. To install on a Claude account or Claude Code instance, copy
the folder into your skills directory or upload the packaged `.skill` file via the
Claude UI. See [Anthropic's skill documentation](https://docs.claude.com) for the
current installation procedure for your environment.

The skill expects `present_files` to be available for delivering the generated
package.

## Repository layout

```
project-design/
├── LICENSE                                       MIT
├── README.md                                     this file
├── SKILL.md                                      skill manifest + body
└── references/
    ├── agent-instruction-patterns.md
    ├── agent-loop-templates.md
    ├── bootstrap-artifacts.md
    ├── brainstorming-techniques.md
    ├── ci-cd-templates.md
    ├── document-templates.md
    ├── engineering-handbook-templates.md
    ├── exploration-map.md
    └── silent-assessment.md
```

## Contributing

Issues and PRs are welcome. For non-trivial changes, open an issue first to discuss.

## License

MIT — see [LICENSE](LICENSE).
