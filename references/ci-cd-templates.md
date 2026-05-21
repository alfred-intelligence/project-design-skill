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
