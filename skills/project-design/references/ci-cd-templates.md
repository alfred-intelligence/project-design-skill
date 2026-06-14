# CI/CD Templates

GitHub Actions workflows and design choices per stack. Paste into `06-ci-cd-plan.md` or
directly into `bootstrap/.github/workflows/`.

---

## General ci.yml structure

Every `ci.yml` follows the same skeleton: checkout → setup → cache → lint → typecheck
→ test → build.

```yaml
name: CI

on:
  push:
    branches: ['**']
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # Stack-specific steps ⇣
```

---

## Go

### bootstrap/.github/workflows/ci.yml

```yaml
name: CI

on:
  push:
    branches: ['**']
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
          cache: true

      - name: Verify go.mod is tidy
        run: |
          go mod tidy
          git diff --exit-code go.mod go.sum

      - name: Lint
        uses: golangci/golangci-lint-action@v6
        with:
          version: latest

      - name: Test
        run: go test -race -coverprofile=coverage.out ./...

      - name: Build
        run: go build ./...

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage.out
```

### bootstrap/.golangci.yml

```yaml
run:
  timeout: 5m

linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - gofmt
    - goimports
    - misspell
    - revive
    - gosec

issues:
  exclude-use-default: false
```

### bootstrap/.github/workflows/release.yml (with GoReleaser)

```yaml
name: Release

on:
  push:
    tags: ['v*.*.*']

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-go@v5
        with:
          go-version-file: go.mod

      - uses: goreleaser/goreleaser-action@v6
        with:
          version: latest
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### bootstrap/.goreleaser.yaml (skeleton)

```yaml
version: 2

before:
  hooks:
    - go mod tidy

builds:
  - env: [CGO_ENABLED=0]
    goos: [linux, darwin, windows]
    goarch: [amd64, arm64]

archives:
  - format: tar.gz
    name_template: >-
      {{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}
    format_overrides:
      - goos: windows
        format: zip

changelog:
  use: github
  sort: asc
  filters:
    exclude: ['^docs:', '^test:', '^chore:']
```

---

## JavaScript/TypeScript (Node)

### bootstrap/.github/workflows/ci.yml

```yaml
name: CI

on:
  push:
    branches: ['**']
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: npm

      - run: npm ci

      - name: Lint
        run: npm run lint

      - name: Typecheck
        run: npm run typecheck

      - name: Test
        run: npm test -- --coverage

      - name: Build
        run: npm run build

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
```

### bootstrap/commitlint.config.cjs

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'subject-case': [2, 'never', ['start-case', 'pascal-case', 'upper-case']],
    'header-max-length': [2, 'always', 72],
  },
};
```

### bootstrap/.github/workflows/release.yml (release-please)

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          release-type: node
          config-file: release-please-config.json
```

### bootstrap/release-please-config.json

```json
{
  "release-type": "node",
  "packages": {
    ".": {
      "release-type": "node",
      "changelog-sections": [
        { "type": "feat", "section": "Features" },
        { "type": "fix", "section": "Bug Fixes" },
        { "type": "perf", "section": "Performance" },
        { "type": "docs", "section": "Documentation", "hidden": false }
      ]
    }
  },
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json"
}
```

---

## Python

### bootstrap/.github/workflows/ci.yml

```yaml
name: CI

on:
  push:
    branches: ['**']
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version-file: pyproject.toml
          cache: pip

      - run: pip install -e ".[dev]"

      - name: Lint
        run: ruff check .

      - name: Format check
        run: ruff format --check .

      - name: Typecheck
        run: mypy .

      - name: Test
        run: pytest --cov --cov-report=xml

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage.xml
```

### bootstrap/ruff.toml

```toml
target-version = "py312"
line-length = 100

[lint]
select = [
  "E", "W",       # pycodestyle
  "F",            # pyflakes
  "I",            # isort
  "N",            # pep8-naming
  "UP",           # pyupgrade
  "B",            # flake8-bugbear
  "S",            # flake8-bandit
  "RUF",          # ruff-specific
]
ignore = ["S101"] # asserts are fine in tests

[lint.per-file-ignores]
"tests/**" = ["S"]
```

---

## Rust

### bootstrap/.github/workflows/ci.yml

```yaml
name: CI

on:
  push:
    branches: ['**']
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

env:
  CARGO_TERM_COLOR: always
  RUSTFLAGS: -D warnings

jobs:
  ci:
    name: fmt + clippy + test + build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Rust toolchain
        uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt, clippy

      - name: Cache cargo
        uses: Swatinem/rust-cache@v2

      - name: Format
        run: cargo fmt --all -- --check

      - name: Clippy
        run: cargo clippy --all-targets --all-features -- -D warnings

      - name: Test
        run: cargo test --all-features --no-fail-fast

      - name: Build (release)
        run: cargo build --release --all-features

  deny:
    name: cargo-deny (license + advisory)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: EmbarkStudios/cargo-deny-action@v2
        with:
          command: check
          arguments: --all-features
```

### bootstrap/rustfmt.toml

```toml
edition = "2021"
max_width = 100
use_small_heuristics = "Default"
imports_granularity = "Module"
group_imports = "StdExternalCrate"
```

### bootstrap/clippy.toml

```toml
# Thresholds for clippy. Generous defaults; tighten as needed.
cognitive-complexity-threshold = 30
type-complexity-threshold = 250
too-many-arguments-threshold = 8
```

### bootstrap/deny.toml

```toml
[graph]
all-features = true

[advisories]
db-path = "~/.cargo/advisory-db"
db-urls = ["https://github.com/rustsec/advisory-db"]
yanked = "deny"
ignore = []

[licenses]
allow = [
  "MIT",
  "Apache-2.0",
  "Apache-2.0 WITH LLVM-exception",
  "BSD-2-Clause",
  "BSD-3-Clause",
  "ISC",
  "Unicode-DFS-2016",
  "MPL-2.0",
  "Zlib",
]
confidence-threshold = 0.93

[bans]
multiple-versions = "warn"
wildcards = "deny"

[sources]
unknown-registry = "deny"
unknown-git = "warn"
allow-git = []
```

### bootstrap/.github/workflows/release.yml — release-please + binary build

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release-please:
    name: release-please
    runs-on: ubuntu-latest
    outputs:
      release_created: ${{ steps.release.outputs.release_created }}
      tag_name: ${{ steps.release.outputs.tag_name }}
    steps:
      - uses: googleapis/release-please-action@v4
        id: release
        with:
          config-file: .github/release-please-config.json
          manifest-file: .github/release-please-manifest.json

  # Activated when the first release ships binaries.
  # build-release-binary:
  #   name: Build release binary
  #   needs: release-please
  #   if: needs.release-please.outputs.release_created == 'true'
  #   strategy:
  #     matrix:
  #       target:
  #         - x86_64-unknown-linux-gnu
  #         - aarch64-unknown-linux-gnu
  #         - x86_64-apple-darwin
  #         - aarch64-apple-darwin
  #         - x86_64-pc-windows-msvc
  #   runs-on: ${{ matrix.os }}
  #   steps:
  #     - uses: actions/checkout@v4
  #     - uses: dtolnay/rust-toolchain@stable
  #       with:
  #         targets: ${{ matrix.target }}
  #     - uses: Swatinem/rust-cache@v2
  #     - run: cargo build --release --target ${{ matrix.target }}
  #     - uses: softprops/action-gh-release@v2
  #       with:
  #         tag_name: ${{ needs.release-please.outputs.tag_name }}
  #         files: target/${{ matrix.target }}/release/<binary-name>*
```

### bootstrap/.github/release-please-config.json (Rust)

```json
{
  "release-type": "rust",
  "packages": {
    ".": {
      "release-type": "rust",
      "changelog-sections": [
        { "type": "feat", "section": "Features" },
        { "type": "fix", "section": "Bug Fixes" },
        { "type": "perf", "section": "Performance" },
        { "type": "docs", "section": "Documentation", "hidden": false }
      ]
    }
  },
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json"
}
```

### Note on cargo-dist

For multi-platform binary distribution, `cargo-dist` is an alternative to a manual
matrix build in release.yml. Adopt it when the first release ships binaries, not
before.

---

## Shell

### bootstrap/.github/workflows/ci.yml

```yaml
name: CI

on:
  push:
    branches: ['**']
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Shellcheck
        uses: ludeeus/action-shellcheck@master
        with:
          severity: warning

      - name: Bats tests
        if: hashFiles('tests/**/*.bats') != ''
        run: |
          sudo apt-get install -y bats
          bats tests/
```

---

## Static site / deploy (render → deploy → smoke)

**Pairs with:** delivery model = deployment, release pattern = Deploy-as-release. The
artifact `ci.yml` above (lint/test/build) is replaced by render → deploy → smoke. There is
no `release.yml` — the deploy *is* the release.

### bootstrap/.github/workflows/deploy.yml

```yaml
name: Deploy

on:
  push:
    branches: [main]          # or a dedicated `deploy` branch fed by a CMS export

permissions:
  contents: read

concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: |
          # render the site (npm run build, hugo, astro build) — a no-op if already static
          echo "build site into ./public"

      - name: Deploy
        run: |
          # ship it (wrangler pages deploy ./public, rsync to host, kubectl apply)
          echo "deploy ./public to target"

      - name: Smoke check
        run: |
          # the gate that turns a deploy into a release
          set -euo pipefail
          base="https://example.com"
          for path in / /a-known-post/ /a-known-category/; do
            code=$(curl -fsS -o /dev/null -w '%{http_code}' "$base$path")
            test "$code" = "200" || { echo "FAIL $path -> $code"; exit 1; }
          done
          echo "smoke ok"
```

**Notes:**
- For a CMS-authored static site the publish pipeline exports to a `deploy` branch; this
  workflow watches that branch. Backend availability does not gate the public deploy.
- Rollback = redeploy the previous good commit, or use the host's deployment history.

---

## Infrastructure / IaC (validate → plan → apply)

**Pairs with:** delivery model = infra/config, release pattern = Apply-as-release. `plan`
runs on PRs (read-only, safe to review); `apply` runs on merge, gated by a protected
environment with a required reviewer.

### bootstrap/.github/workflows/iac.yml

```yaml
name: IaC

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read
  pull-requests: write        # to post the plan back onto the PR

concurrency:
  group: iac-${{ github.ref }}
  cancel-in-progress: false   # never cancel an in-flight apply

jobs:
  plan:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate + plan
        run: |
          # terraform fmt -check && terraform validate && terraform plan -out plan.tfout
          echo "validate and plan"

  apply:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: prod          # required reviewer configured in repo settings
    steps:
      - uses: actions/checkout@v4
      - name: Apply
        run: |
          # terraform apply -auto-approve
          echo "apply"
      - name: Post-apply check
        run: |
          # drift or health check confirming desired state
          echo "verify"
```

**Notes:**
- The `apply` job's `environment: prod` is the approval gate — pair it with a required
  reviewer (see `bootstrap-artifacts.md` § branch-protection / repo-settings).
- A remote, locked state backend is assumed; never commit state to the repo.

---

## Security scanning (shared)

### bootstrap/.github/workflows/codeql.yml

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1' # Mondays 06:00 UTC

permissions:
  actions: read
  contents: read
  security-events: write

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        language: [go]   # adjust per stack
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/autobuild@v3
      - uses: github/codeql-action/analyze@v3
```

---

## Self-hosted runners (if relevant)

```yaml
jobs:
  ci:
    runs-on: [self-hosted, lab, linux]
    # ...
```

**Requirements:**
- Runner host hardened per the project's security policy.
- Only trusted workflows may run (PRs from forks → not self-hosted).
- Runner auto-update enabled.
- Logs shipped to a central log aggregator.

---

## Environment definitions — typical settings

```markdown
| Environment | URL pattern | Auto-deploy | Approvals | Secrets |
|-------------|-------------|-------------|-----------|---------|
| dev | dev.example.com | push to `develop` | no | dev-secrets |
| staging | staging.example.com | push to `main` | no | staging-secrets |
| prod | example.com | tag `v*.*.*` | required | prod-secrets |
```

GitHub Environments: configure under repo Settings → Environments. Required reviewers
on `prod` are recommended even at team level.
