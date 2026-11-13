---
name: project-launchpad
description: Turns a software idea into a ready-to-build GitHub repository in one go — scaffolded project, working dev/test/lint commands, GitHub Actions CI, a CD pipeline that deploys, repo hygiene (Dependabot, templates, branch protection), an MVP backlog of issues, and optionally a daily AI agent (Claude or Gemini) that opens issues and PRs. Use this whenever the user wants to start, set up, bootstrap, scaffold, kick off, or "get going" with a new project, app, web app, API, CLI, bot, or other software idea, or says setup/boilerplate/CI is slowing them down — even if they don't mention GitHub Actions or CI/CD.
---

# Project Launchpad

Setting up a project is the part where good ideas die: picking tools, wiring CI, figuring out deployment, writing config. This skill removes that friction. The goal is that at the end the user has a repo where **the first real feature is the only thing left to do**: `git push` runs tests, merging to main deploys, and there's a backlog of small issues to start on.

Optimize for momentum over perfection. Pick sensible, boring, well-supported defaults and explain them in one line each — the user can change them later. Don't make them answer 15 questions.

## Phase 1: Intake (one round of questions, max)

Ask everything you need in a single message, offering a recommended default for each so the user can just say "defaults are fine". If they've already answered some in their request, don't re-ask.

1. **The idea** – one or two sentences: what it does and who it's for.
2. **Type** – web app, API/backend, CLI, bot, mobile, library, something else.
3. **Stack** – their preferred language/framework, or "recommend one". If recommending, favour what they already know (check memory/context) and mainstream options with good CI and hosting support.
4. **Where it runs** – deployment target, or "recommend one" (see `references/ci-cd.md` for the decision table).
5. **Repo** – name, public or private.
6. **Daily AI agent** – none / Claude / Gemini (explained in Phase 6). Default: none for now, can add later.

Then write a short **project brief** before building anything, and confirm it with the user:

```
Name: …
Pitch: one sentence
MVP (3–6 features): the smallest version that's actually useful
Not in MVP: things deliberately left for later
Stack: … | Hosting: … | Agent: …
```

The brief matters more than it looks: it keeps scope small, becomes the README intro and the MVP issues, and gives any AI agent the context it needs to make useful contributions instead of random ones.

## Phase 2: Scaffold the project

Use the framework's official generator when one exists (e.g. `npm create vite@latest`, `uv init`, `cargo new`, `dotnet new`) rather than hand-writing boilerplate — it stays current and idiomatic. Then add, in every project:

- **One-command workflows**: `dev`, `test`, `lint`, `format`, `build` exposed the same way (package.json scripts, a `Makefile`, or `justfile`). CI calls these same commands, so "works locally" means "works in CI".
- **Lint + format + type-check** with the ecosystem's standard tools (see `references/ci-cd.md`), configured, and passing on the scaffold.
- **A real test setup with at least one passing test**, so the test pipeline is proven before any feature exists.
- **`.env.example`** listing every config value (no real secrets), with `.env` in `.gitignore`.
- **README.md**: pitch, how to run/test locally, how deployment works, link to the brief.
- **`AGENTS.md`** (and a `CLAUDE.md` that just says to read AGENTS.md, if useful): stack, commands, conventions, folder layout, "always run tests before opening a PR". Any AI coding tool — the daily agent, Claude Code, Gemini CLI — reads this to work correctly in the repo.
- `.gitignore`, `.editorconfig`, and a LICENSE if the repo is public (ask; default MIT).

Run the dev, lint, test and build commands yourself before moving on if you have an environment that can. A scaffold that doesn't pass its own CI is worse than none.

## Phase 3: CI with GitHub Actions

Create `.github/workflows/ci.yml` that runs on pull requests and pushes to main:

- Install with dependency **caching**; pin the runtime version.
- Run lint → type-check → test → build, using the same commands as local.
- `concurrency` with `cancel-in-progress` so pushes don't queue up.
- Minimal `permissions:` (`contents: read` unless more is needed).
- Use current major versions of official actions (`actions/checkout`, `actions/setup-node`, etc.). If you have web access, check the latest major version instead of relying on memory.

Templates per stack are in `references/ci-cd.md`.

## Phase 4: CD (deployment)

Create `.github/workflows/deploy.yml` that deploys when CI passes on main, and preview deploys for PRs when the host supports it.

- Secrets (API tokens, deploy keys) go in **GitHub repository secrets** — never in the repo, never in the workflow file. Tell the user exactly which secrets to create, where to get each value, and the exact name to use. Don't ask them to paste secrets into the chat.
- Use the host's official GitHub Action or CLI.
- Use a GitHub **environment** (`production`) so deploys are visible and can require approval later.
- If the host does git-based auto-deploys natively (e.g. Vercel, Netlify, Cloudflare Pages connected to the repo), say so: that may replace the deploy workflow entirely, and CI still gates merges.

## Phase 5: Repo hygiene and backlog

- **Dependabot** (`.github/dependabot.yml`) for the package ecosystem and for GitHub Actions, weekly, grouped.
- **PR template** with a checklist (tests added, docs updated) and **issue templates** for bug and feature.
- **Labels**: `mvp`, `bug`, `enhancement`, `good-first-task`, and `ai-ready` / `ai-generated` if an agent is set up.
- **MVP backlog**: one issue per MVP feature from the brief, each small enough for one PR, with acceptance criteria. This turns "where do I start?" into "pick issue #1".
- **Branch protection** on main: require PRs and passing CI. This needs repo admin, so do it via `gh api` if available, otherwise give the user the exact settings to click.

## Phase 6: Optional daily AI agent

If the user wants it, add a scheduled workflow that runs Claude (`anthropics/claude-code-action`) or Gemini (`google-github-actions/run-gemini-cli`) once a day. Read `references/ai-agent.md` before writing it; it has the templates and the guardrails.

The design that works well: the agent **works from the backlog instead of inventing work**. Each run it picks up to one issue labeled `ai-ready`, implements it on a branch, runs the tests, and opens a PR linked to the issue. Optionally it also files up to N issues for bugs or improvements it notices, labeled `ai-generated` for the user to triage. It never merges and never pushes to main. That way the user stays in control and API costs stay predictable.

## Phase 7: Create the repo and hand off

How you deliver depends on the environment:

- **Shell with `gh` and git** (e.g. Claude Code): create the repo, push, set secrets the user provides through `gh secret set` (they type the value, not you), create labels and issues, set branch protection, then watch the first CI run go green with `gh run watch`.
- **No shell access to their machine** (e.g. claude.ai chat): build the full project in the sandbox, verify it there, and give it to the user as a zip with a short `setup.sh` (or exact commands) that creates the repo with `gh`, pushes, creates labels and issues. If a browser tool is connected and they ask you to, you can create the repo and files through GitHub's web UI instead.

Finish with a short handoff:
- Repo link, and what happens on push / PR / merge.
- Secrets still to add (name + where to get it), if any.
- The one next step: "Start with issue #1: …".

## Things to avoid

- Over-engineering: no Kubernetes, microservices, or monorepo tooling for an idea that doesn't exist yet. Add it when it's needed.
- Unverified config: if you couldn't run it, say which parts are unverified.
- Silent choices: every default gets a one-line "why", so the user can override it.
