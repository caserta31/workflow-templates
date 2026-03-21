# Workflow Templates

Reusable GitHub Action workflows for the code factory. Each project repo uses thin caller workflows that reference these templates.

## Available Workflows

### `risk-policy-gate.yml`
Reads `risk-contract.json` from the calling repo, classifies changed files by risk tier (high/medium/low), posts a PR comment with the classification, and adds labels for high-risk changes. No LLM calls — pure file matching. Runs in < 30 seconds.

### `auto-review.yml`
Runs Claude Code as an automated code reviewer on every PR. Reads AGENTS.md, CLAUDE.md, and risk-contract.json for project context. Posts inline comments and a summary verdict.

## Usage

Add thin caller workflows to your project repo:

```yaml
# .github/workflows/risk-policy-gate.yml
name: Risk Policy Gate
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  gate:
    uses: proc015/workflow-templates/.github/workflows/risk-policy-gate.yml@main

# .github/workflows/auto-review.yml
name: Auto Code Review
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    needs: [gate]  # Only review if gate passes
    uses: proc015/workflow-templates/.github/workflows/auto-review.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Requirements

Each project repo needs:
- `risk-contract.json` — risk tier definitions and merge policy
- `AGENTS.md` — agent-readable project map
- `CLAUDE.md` — style and convention guide
- `ANTHROPIC_API_KEY` secret (for auto-review)
