# Bootstrap Artifacts

Importable repo files for GitHub. Always generated when the skill runs. Content scales
with strictness level (`solo` / `solo+contrib` / `team`) and public/private status.

---

## Overview

```
bootstrap/
├── LICENSE
├── LICENSE-docs              (if doc license is not "All Rights Reserved")
├── README.md
├── IMPORT.md                (always — gh commands to import the artifacts)
├── .gitignore
├── .editorconfig
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md       (if public or team)
├── SECURITY.md              (if public or team)
└── .github/
    ├── labels.json
    ├── branch-protection.json
    ├── repo-settings.json
    ├── dependabot.yml
    ├── PULL_REQUEST_TEMPLATE.md
    ├── ISSUE_TEMPLATE/
    │   ├── bug_report.yml
    │   ├── feature_request.yml
    │   └── config.yml
    └── workflows/
        ├── ci.yml
        └── release.yml
```

Plus stack-specific files in the `bootstrap/` root (see `ci-cd-templates.md`).

---

## Delivery-model additions

The set above assumes a versioned artifact (library, CLI, binary). Two delivery models
swap a few files; everything else — LICENSE, README, labels, templates, branch protection,
CoC/SECURITY — is unchanged.

### Static site / deployment

- **Workflows:** `deploy.yml` (render → deploy → smoke) replaces `ci.yml` + `release.yml`.
  See `ci-cd-templates.md` § Static site / deploy. There is no release workflow.
- **.gitignore:** add the generator's output and cache:
  ```gitignore
  # Static site
  public/
  dist/
  .cache/
  ```
- **dependabot.yml:** keep `github-actions`; add the generator's ecosystem (`npm` for
  Astro/Eleventy; none for raw static).
- **README:** drop version badges from Status — a deploy or uptime link fits better.
- **Branch model:** if a CMS exports to a `deploy` branch, that branch is the deploy
  trigger and is machine-written, not protected the way `main` is.

### Infrastructure / IaC

- **Workflows:** `iac.yml` (validate → plan → apply) replaces `ci.yml` + `release.yml`.
  See `ci-cd-templates.md` § Infrastructure / IaC.
- **Environment gate:** configure a protected `prod` environment with a required reviewer
  (repo Settings → Environments); the `apply` job runs there.
- **.gitignore:** never commit state or secrets:
  ```gitignore
  # IaC
  *.tfstate
  *.tfstate.*
  .terraform/
  plan.tfout
  *.tfvars        # if they hold secrets — commit *.tfvars.example instead
  ```
- **dependabot.yml:** add `terraform` (or the relevant provider ecosystem) alongside
  `github-actions`.
- **No SemVer:** environment tags (`prod-YYYY-MM-DD`) if any, not version tags.

---

## Import commands

### Apply labels

```bash
# Requires: gh CLI authenticated against the right repo
gh label list --json name --jq '.[].name' | xargs -I {} gh label delete {} --yes
jq -c '.[]' bootstrap/.github/labels.json | while read -r label; do
  name=$(echo "$label" | jq -r .name)
  color=$(echo "$label" | jq -r .color)
  desc=$(echo "$label" | jq -r .description)
  gh label create "$name" --color "$color" --description "$desc"
done
```

Alternatively, use [`github-label-sync`](https://github.com/Financial-Times/github-label-sync):
```bash
npx github-label-sync --access-token "$GH_TOKEN" --labels bootstrap/.github/labels.json owner/repo
```

### Apply branch protection

```bash
gh api -X PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/{owner}/{repo}/branches/main/protection \
  --input bootstrap/.github/branch-protection.json
```

### Apply repo settings

```bash
gh api -X PATCH \
  -H "Accept: application/vnd.github+json" \
  /repos/{owner}/{repo} \
  --input bootstrap/.github/repo-settings.json
```

---

## IMPORT.md (standard file in bootstrap)

Always generated at `bootstrap/IMPORT.md`. The operator's guide to applying the
artifacts against a fresh GitHub repo. All `gh` commands collected in one place.

````markdown
# Bootstrap — Import guide

Step-by-step import of the artifacts in this folder into a fresh GitHub repo.

## Prerequisites

- `gh` CLI installed and authenticated (`gh auth status` should be green)
- `jq` installed
- Repo already created (private or public) on GitHub

## Step 1 — Copy files to the repo root

```bash
cd <your-repo-folder>
cp -r <path-to-bootstrap>/. .
git add .
git commit -m "chore: initial bootstrap from project-design"
git push
```

## Step 2 — Import labels

```bash
# Clear default labels and import our set
gh label list --json name --jq '.[].name' | xargs -I {} gh label delete {} --yes 2>/dev/null
jq -c '.[]' .github/labels.json | while read -r label; do
  name=$(echo "$label" | jq -r .name)
  color=$(echo "$label" | jq -r .color)
  desc=$(echo "$label" | jq -r .description)
  gh label create "$name" --color "$color" --description "$desc"
done
```

## Step 3 — Apply branch protection on main

```bash
gh api -X PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/<owner>/<repo>/branches/main/protection \
  --input .github/branch-protection.json
```

## Step 4 — Apply repo settings

```bash
gh api -X PATCH \
  -H "Accept: application/vnd.github+json" \
  /repos/<owner>/<repo> \
  --input .github/repo-settings.json
```

## Step 5 — Verify

```bash
# Labels
gh label list

# Branch protection
gh api /repos/<owner>/<repo>/branches/main/protection

# Repo settings
gh repo view --json hasIssuesEnabled,squashMergeAllowed,deleteBranchOnMerge
```

## Step 6 — First PR

Create a test branch, make a trivial commit, open a PR, check that CI runs and that
branch protection blocks direct push to `main`.

```bash
git checkout -b chore/bootstrap-test
echo "" >> README.md
git commit -am "chore: bootstrap test"
git push -u origin chore/bootstrap-test
gh pr create --fill
```

If all steps succeed, the repo is bootstrapped and ready for development per
05-engineering-handbook and 07-agent-loop.
````

---

## LICENSE

Content depends on the choice in 05. Take canonical text from
[choosealicense.com](https://choosealicense.com/) or directly from the Open Source
Initiative. Fill in year and copyright holder. For a private repo:

```
Copyright (c) [year] [copyright holder]

All rights reserved. This software is proprietary and confidential.
Unauthorized copying, modification, distribution, or use is strictly prohibited.
```

Use the placeholder scheme from SKILL.md § Identity neutrality (`operator`, `acme-org`)
until the confidentiality checklist has confirmed the real holder name.

---

## LICENSE-docs

A separate license file for documentation, generated by default when the doc license
is not "All Rights Reserved". See `references/engineering-handbook-templates.md`
§ Documentation license for the source → doc license mapping.

Take canonical text from [creativecommons.org/licenses](https://creativecommons.org/licenses/).

**CC-BY 4.0** — for projects with permissive source licenses (MIT, Apache-2.0,
BSD-*, ISC).
- URL: https://creativecommons.org/licenses/by/4.0/legalcode.txt
- Short attribution notice on each page is recommended but not required.

**CC-BY-SA 4.0** — for projects with copyleft source licenses (GPL-*, AGPL-3.0,
MPL-2.0, LGPL-*).
- URL: https://creativecommons.org/licenses/by-sa/4.0/legalcode.txt
- Derivative works must be licensed under CC-BY-SA 4.0 as well.

**Proprietary doc license** (when source is BSL/SSPL/All Rights Reserved):

No `LICENSE-docs` file is generated. Add a notice to `README.md` instead:

```
Documentation © [year] [copyright holder]. All rights reserved.
```

### Scope notice in README

When a `LICENSE-docs` file is generated, the README skeleton's License section
lists both files. See § README.md (skeleton) below.

---

## README.md (skeleton)

```markdown
# [Project name]

> [One sentence about what the project is.]

## Status

[Badges: CI, license, version]

## Features

- [Item 1]
- [Item 2]

## Getting started

```bash
# Installation
[command]

# Quickstart
[command]
```

## Documentation

See [`docs/`](docs/) or [`01-whitepaper.md`](01-whitepaper.md) for design.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Source code: [License name] — see [LICENSE](LICENSE).
Documentation: [Doc license name] — see [LICENSE-docs](LICENSE-docs).

Documentation covers `README.md`, `docs/`, `{project}-design/`, and other top-level
`.md` files. `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, and `SECURITY.md` have their
own licenses (Contributor Covenant is CC-BY 4.0).
```

---

## .gitignore (stack-specific)

Generate based on stack. Use [gitignore.io](https://www.toptal.com/developers/gitignore)
or `github/gitignore` templates as the source.

**Common base lines:**

```gitignore
# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/
*.swp

# Secrets
.env
.env.local
*.pem

# Build artifacts
dist/
build/
node_modules/
target/
```

---

## .editorconfig

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.go]
indent_style = tab

[*.py]
indent_size = 4

[Makefile]
indent_style = tab

[*.md]
trim_trailing_whitespace = false
```

---

## CONTRIBUTING.md (per level)

### Solo (short)

```markdown
# Contributing to [Project]

This project is primarily developed by [maintainer]. External contributions are not
actively sought, but issues and PRs are read.

## Issues

Open an issue if you find a bug or have an idea.

## PRs

Large changes: open an issue first for discussion.
Small fixes: a direct PR is fine.
```

### Solo+contrib (standard)

```markdown
# Contributing to [Project]

Thanks for the interest. Here is the process:

## Before you start

- Read [README.md](README.md) and [01-whitepaper.md](01-whitepaper.md).
- Large changes: open an issue first for discussion.

## Development environment

```bash
git clone <repo>
cd <repo>
[setup commands]
```

## Commit conventions

[Conventional Commits](https://www.conventionalcommits.org/). See
[05-engineering-handbook.md].

## PR process

1. Fork and create a feature branch (`feat/<scope>-<description>`).
2. Run tests locally: `[test command]`.
3. Open a PR against `main` with a Conventional Commits title.
4. CI must be green.
5. Wait for review.

## Code of Conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
```

### Team (extensive)

```markdown
# Contributing to [Project]

[As above, plus:]

## Roles and responsibilities

| Role | Responsibility |
|------|----------------|
| Maintainer | Final approval on PRs, releases |
| Reviewer | Review PRs |
| Contributor | Submit PRs |

## Design documents

Larger changes (new features, architectural shifts) require an RFC in `docs/rfc/`
before implementation.

## Review SLA

- Initial review within 2 work days.
- Follow-up review within 1 work day after the latest push.

## Release routine

See [05-engineering-handbook.md § Release process].
```

---

## CODE_OF_CONDUCT.md

Use [Contributor Covenant 2.1](https://www.contributor-covenant.org/) as the default.
Generate the full text — read the current version from the source.

Add contact information for reporting: `[email/Discord/GitHub Discussions]`. Use
placeholder addresses (`operator@example.com`) until the confidentiality checklist
confirms a real one.

---

## SECURITY.md

```markdown
# Security policy

## Supported versions

| Version | Support |
|---------|---------|
| latest  | Yes |
| < latest | No |

## Reporting a vulnerability

**Do NOT report security issues via GitHub Issues.**

Send instead to [security@example.com] with:
- Description of the vulnerability
- Reproduction steps
- Impact
- Optional proposed fix

We acknowledge receipt within 48 hours and provide an initial assessment within 7 days.

## Disclosure policy

We follow [coordinated disclosure](https://en.wikipedia.org/wiki/Coordinated_vulnerability_disclosure).
Thanks for helping keep the project safe.
```

---

## .github/labels.json

```json
[
  { "name": "bug", "color": "d73a4a", "description": "Something is not working" },
  { "name": "enhancement", "color": "a2eeef", "description": "New feature or improvement" },
  { "name": "documentation", "color": "0075ca", "description": "Documentation improvements" },
  { "name": "good first issue", "color": "7057ff", "description": "Good for new contributors" },
  { "name": "help wanted", "color": "008672", "description": "Extra attention needed" },
  { "name": "duplicate", "color": "cfd3d7", "description": "Duplicate of another issue/PR" },
  { "name": "invalid", "color": "e4e669", "description": "Not a valid issue" },
  { "name": "wontfix", "color": "ffffff", "description": "Will not be fixed" },
  { "name": "question", "color": "d876e3", "description": "More information needed" },
  { "name": "dependencies", "color": "0366d6", "description": "Dependency updates" },
  { "name": "security", "color": "ee0701", "description": "Security-related" },
  { "name": "breaking-change", "color": "b60205", "description": "Breaks backwards compatibility" },
  { "name": "needs-triage", "color": "fbca04", "description": "Not yet triaged" }
]
```

---

## .github/branch-protection.json

### Solo

```json
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": null,
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "required_linear_history": true,
  "required_conversation_resolution": true
}
```

### Solo+contrib

```json
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "required_linear_history": true,
  "required_conversation_resolution": true
}
```

### Team

```json
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci", "codeql"]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 2,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true,
    "require_last_push_approval": true
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "required_linear_history": true,
  "required_conversation_resolution": true
}
```

---

## .github/repo-settings.json

```json
{
  "has_issues": true,
  "has_projects": false,
  "has_wiki": false,
  "has_discussions": false,
  "allow_squash_merge": true,
  "allow_merge_commit": false,
  "allow_rebase_merge": true,
  "allow_auto_merge": true,
  "delete_branch_on_merge": true,
  "squash_merge_commit_title": "PR_TITLE",
  "squash_merge_commit_message": "PR_BODY"
}
```

---

## .github/dependabot.yml

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: "/"
    schedule:
      interval: weekly
    labels: ["dependencies", "ci"]

  # Add stack-specific entries below ⇣
  # - package-ecosystem: gomod
  #   directory: "/"
  #   schedule: { interval: weekly }
  #
  # - package-ecosystem: npm
  #   directory: "/"
  #   schedule: { interval: weekly }
  #
  # - package-ecosystem: pip
  #   directory: "/"
  #   schedule: { interval: weekly }
```

---

## .github/PULL_REQUEST_TEMPLATE.md

```markdown
## What

<!-- Short description of the change -->

## Why

<!-- Motivation. Link to an issue if applicable. -->
Closes #

## How

<!-- How has it been tested? Manual steps if relevant. -->

## Migrations / config changes

<!-- Any migration, env update, or config change needed? -->

## Checklist

- [ ] Tests added or updated
- [ ] Documentation updated
- [ ] Conventional Commits-formatted PR title
- [ ] CI is green
```

---

## .github/ISSUE_TEMPLATE/bug_report.yml

```yaml
name: Bug report
description: Report a bug
labels: ["bug", "needs-triage"]
body:
  - type: markdown
    attributes:
      value: Thanks for taking the time to report. Be concrete and include reproduction.

  - type: textarea
    id: description
    attributes:
      label: Description
      description: What happened? What did you expect?
    validations:
      required: true

  - type: textarea
    id: reproduction
    attributes:
      label: Reproduction
      description: Steps to reproduce, preferably minimal
      placeholder: |
        1. ...
        2. ...
        3. ...
    validations:
      required: true

  - type: textarea
    id: environment
    attributes:
      label: Environment
      placeholder: |
        - Version:
        - OS:
        - [Stack-specific fields]
    validations:
      required: true

  - type: textarea
    id: logs
    attributes:
      label: Logs
      render: shell
```

---

## .github/ISSUE_TEMPLATE/feature_request.yml

```yaml
name: Feature request
description: Propose a new feature or improvement
labels: ["enhancement", "needs-triage"]
body:
  - type: textarea
    id: problem
    attributes:
      label: Problem
      description: What problem does this solve?
    validations:
      required: true

  - type: textarea
    id: proposal
    attributes:
      label: Proposal
      description: What would the solution look like?
    validations:
      required: true

  - type: textarea
    id: alternatives
    attributes:
      label: Alternatives
      description: Other solutions considered

  - type: checkboxes
    id: contribution
    attributes:
      label: Contribute
      options:
        - label: I am willing to submit a PR for this
```

---

## .github/ISSUE_TEMPLATE/config.yml

```yaml
blank_issues_enabled: false
contact_links:
  - name: Security vulnerability
    url: https://github.com/{owner}/{repo}/security/advisories/new
    about: Report security issues privately. Do NOT use issues.
  - name: Questions and discussion
    url: https://github.com/{owner}/{repo}/discussions
    about: General questions and discussion
```
