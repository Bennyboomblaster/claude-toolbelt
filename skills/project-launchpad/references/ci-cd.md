# CI/CD reference

Contents: 1. Stack defaults · 2. CI template · 3. Hosting decision table · 4. Deploy examples

Versions below are reasonable defaults; check current major versions if web access is available.

## 1. Stack defaults

| Stack | Package/run | Lint/format | Types | Tests |
|---|---|---|---|---|
| TypeScript web (React/Vite, Next.js) | pnpm or npm | ESLint + Prettier (or Biome) | `tsc --noEmit` | Vitest (+ Playwright for e2e later) |
| Node API | pnpm/npm | ESLint + Prettier (or Biome) | `tsc --noEmit` | Vitest |
| Python | uv | Ruff (lint + format) | mypy or pyright | pytest |
| Go | go modules | gofmt + golangci-lint | built-in | `go test ./...` |
| Rust | cargo | rustfmt + clippy | built-in | `cargo test` |
| C# / .NET | dotnet | `dotnet format` | built-in | xUnit |

## 2. CI template (adapt the setup + commands per stack)

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  check:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      # --- Node example ---
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm test
      - run: pnpm build
```

Python swap-in:
```yaml
      - uses: astral-sh/setup-uv@v5
        with: { enable-cache: true }
      - run: uv sync --locked
      - run: uv run ruff check . && uv run ruff format --check .
      - run: uv run pytest
```

Go: `actions/setup-go` with `go-version-file: go.mod` (caching built in), then `go vet ./...`, `golangci-lint` via `golangci/golangci-lint-action`, `go test ./...`.
Rust: `dtolnay/rust-toolchain@stable` + `Swatinem/rust-cache`, then `cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo test`.

## 3. Hosting decision table

| Project | Good default | Why |
|---|---|---|
| Static site / SPA | Cloudflare Pages, Netlify, or GitHub Pages | Free, PR previews (not GH Pages), zero servers |
| Next.js / full-stack JS | Vercel | First-class support, PR previews |
| API / backend / anything in a container | Fly.io, Render, or Railway | Simple deploys from a Dockerfile, managed Postgres options |
| Need a database | Host's managed Postgres, Supabase, or Neon | Don't self-host a DB for an MVP |
| CLI / library | GitHub Releases (+ npm/PyPI/crates.io) | Release workflow on tag push |
| Bot / scheduled job | Fly.io / Render, or GitHub Actions cron if tiny | |

## 4. Deploy examples

Deploy only after CI passes on main:

```yaml
name: Deploy
on:
  workflow_run:
    workflows: [CI]
    types: [completed]
    branches: [main]

permissions:
  contents: read

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    environment: production
    concurrency: deploy-production
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.workflow_run.head_sha }}
      # Fly.io example
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

Alternatives: a single workflow with `deploy` job `needs: check` and `if: github.ref == 'refs/heads/main'` is simpler and fine for small projects.

Container images: build and push to GHCR with `docker/login-action` + `docker/build-push-action`, `permissions: packages: write`.

Libraries/CLIs: consider `googleapis/release-please-action` for automatic changelogs, version bumps, and tagged releases from conventional commits.
