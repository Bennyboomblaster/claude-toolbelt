# Daily AI agent reference

Read this before adding a scheduled Claude or Gemini workflow. Action inputs change between versions; if web access is available, check the action's README for the current inputs before writing the workflow.

## Design

The agent's job each run, in priority order:
1. **Implement**: take the oldest open issue labeled `ai-ready` that has no open PR. Create a branch `ai/issue-<n>`, implement it following AGENTS.md, run lint + tests, open a PR that says `Closes #<n>`, labeled `ai-generated`. At most one per run.
2. **Triage** (optional): skim the code and recent CI failures and open at most 2 issues for concrete bugs or small improvements, labeled `ai-generated` + `needs-triage`. Check existing issues first to avoid duplicates.
3. If there's nothing worth doing, do nothing. An empty run is a good run.

The user decides what's `ai-ready`. New-feature ideas from the agent go into issues for the user to approve, never straight into PRs.

## Guardrails (always include)

- **Never auto-merge, never push to main.** Branch protection enforces this; the PR still has to pass CI and be reviewed.
- **Cost caps**: limit turns (`--max-turns`) and, where supported, budget (e.g. Claude Code's `--max-budget-usd`), plus `timeout-minutes` on the job. Tell the user the rough expected cost per run.
- **Minimal permissions**: `contents: write`, `pull-requests: write`, `issues: write`, nothing else.
- **Trigger only on `schedule` and `workflow_dispatch`.** Don't run this workflow on `pull_request_target`, issue comments, or anything strangers can trigger in a public repo: issue/PR text is untrusted input and could contain prompt injection aimed at the API key and write access.
- **Only act on issues labeled by the owner.** In public repos, anyone can open issues; a label only maintainers can apply (`ai-ready`) is the gate.
- **API key in secrets** (`ANTHROPIC_API_KEY` or `GEMINI_API_KEY`); the user adds it themselves in repo settings or with `gh secret set` (they type it).
- **PRs opened with the default `GITHUB_TOKEN` don't trigger other workflows**, so CI won't run on the agent's PRs. Fix with one of: the official Claude GitHub App (via `/install-github-app` in Claude Code), a GitHub App token, or a fine-grained PAT stored as a secret. Explain the trade-off to the user.

## Claude template (anthropics/claude-code-action@v1)

```yaml
name: Daily AI agent
on:
  schedule:
    - cron: "0 6 * * *"   # 06:00 UTC daily
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write
  issues: write

concurrency:
  group: daily-agent
  cancel-in-progress: false

jobs:
  agent:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
      # + the same runtime setup steps as CI, so the agent can run tests
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # github_token: ${{ steps.app-token.outputs.token }}  # so CI runs on its PRs
          prompt: |
            You are the daily maintenance agent for ${{ github.repository }}.
            Read AGENTS.md first and follow it.
            1. Find the oldest open issue labeled "ai-ready" with no linked open PR.
               If one exists: create branch ai/issue-<number>, implement it, run lint and
               tests until they pass, and open a PR with "Closes #<number>", labeled
               "ai-generated". Do at most one issue.
            2. Then, open at most 2 new issues for concrete bugs or small improvements you
               noticed, labeled "ai-generated" and "needs-triage". Search existing issues
               first; don't create duplicates.
            Never push to main, never merge. If nothing is worth doing, do nothing.
          claude_args: |
            --max-turns 30
            --allowedTools "Bash,Read,Edit,Write,Glob,Grep"
```

## Gemini template (google-github-actions/run-gemini-cli)

Same workflow shell (triggers, permissions, concurrency, checkout, runtime setup), with the agent step swapped:

```yaml
      - uses: google-github-actions/run-gemini-cli@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          gemini_api_key: ${{ secrets.GEMINI_API_KEY }}
          prompt: |
            <same prompt as the Claude template>
```

Gemini CLI uses the `gh` CLI (available on GitHub runners) for issues/PRs; make sure the prompt tells it to. Check the action's README for current input names and tool-permission settings.

## Useful extras (offer, don't force)

- **On-demand**: an `@claude` / `@gemini-cli` mention trigger in comments so the user can ask for changes on any PR. Restrict to the repo owner (`github.event.comment.author_association == 'OWNER'`) in public repos.
- **AI PR review** on every PR (cheap, helpful when the agent opens PRs too).
- **Weekly summary issue**: what merged, what's open, what's blocked.
