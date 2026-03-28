# Workflow Templates

Reusable GitHub Action workflows for the code factory. Each project repo uses thin caller workflows that reference these templates.

---

## Available Workflows

### Language-specific (pick one)

| Workflow | Stack | What it does |
|---|---|---|
| `ci-pipeline.yml` | JS/Node | Install, lint, test for React + Vite / Express repos |
| `ci-pipeline-python.yml` | Python | Install, lint (ruff), test (pytest) for Flask/FastAPI repos |

### Shared (use with any language)

| Workflow | What it does |
|---|---|
| `code-health.yml` | Posts a file-length scorecard on PRs (non-blocking) |
| `risk-policy-gate.yml` | Classifies changed files by risk tier, posts PR comment |
| `auto-review.yml` | Claude-powered code review with inline comments |

---

## Onboarding: JavaScript / Node Repo

For repos like **rivo**, **evaso**, **asola** (React + Vite frontend, Express backend, MongoDB).

### 1. Copy the risk contract

```bash
curl -o risk-contract.json https://raw.githubusercontent.com/caserta31/workflow-templates/main/risk-contract.template.json
```

Edit the file patterns in `risk-contract.json` to match your project structure.

### 2. Add the ANTHROPIC_API_KEY secret

Repo **Settings > Secrets and variables > Actions** → add `ANTHROPIC_API_KEY`.

### 3. Create `.github/workflows/ci.yml`

```yaml
name: CI
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ci:
    uses: caserta31/workflow-templates/.github/workflows/ci-pipeline.yml@main
    # Override defaults if needed:
    # with:
    #   frontend_dir: frontend
    #   backend_dir: api
    #   package_manager: yarn

  code-health:
    uses: caserta31/workflow-templates/.github/workflows/code-health.yml@main
    # with:
    #   extensions: "js,jsx,ts,tsx"

  risk-gate:
    needs: ci
    uses: caserta31/workflow-templates/.github/workflows/risk-policy-gate.yml@main

  review:
    needs: [ci, risk-gate]
    uses: caserta31/workflow-templates/.github/workflows/auto-review.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### CI Pipeline inputs

| Input | Default | Description |
|---|---|---|
| `node_version` | `20` | Node.js version |
| `package_manager` | `npm` | `npm` or `yarn` |
| `has_frontend` | `true` | Run frontend lint/test |
| `has_backend` | `true` | Run backend lint/test |
| `frontend_dir` | `client` | Frontend directory |
| `backend_dir` | `server` | Backend directory |

### Backend-only JS repo

```yaml
jobs:
  ci:
    uses: caserta31/workflow-templates/.github/workflows/ci-pipeline.yml@main
    with:
      has_frontend: false
      backend_dir: .
```

---

## Onboarding: Python Repo

For Flask, FastAPI, or Django repos.

### 1. Copy the risk contract

```bash
curl -o risk-contract.json https://raw.githubusercontent.com/caserta31/workflow-templates/main/risk-contract.python.template.json
```

Edit the file patterns in `risk-contract.json` to match your project structure.

### 2. Add the ANTHROPIC_API_KEY secret

Repo **Settings > Secrets and variables > Actions** → add `ANTHROPIC_API_KEY`.

### 3. Create `.github/workflows/ci.yml`

```yaml
name: CI
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ci:
    uses: caserta31/workflow-templates/.github/workflows/ci-pipeline-python.yml@main
    # Override defaults if needed:
    # with:
    #   python_version: "3.11"
    #   package_manager: poetry
    #   app_dir: src

  code-health:
    uses: caserta31/workflow-templates/.github/workflows/code-health.yml@main
    with:
      extensions: "py"

  risk-gate:
    needs: ci
    uses: caserta31/workflow-templates/.github/workflows/risk-policy-gate.yml@main

  review:
    needs: [ci, risk-gate]
    uses: caserta31/workflow-templates/.github/workflows/auto-review.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### CI Pipeline inputs

| Input | Default | Description |
|---|---|---|
| `python_version` | `3.12` | Python version |
| `package_manager` | `pip` | `pip` or `poetry` |
| `app_dir` | `.` | Application directory |
| `requirements_file` | `requirements.txt` | Requirements file path (pip only) |

---

## Shared Workflow Inputs

### `code-health.yml`

| Input | Default | Description |
|---|---|---|
| `frontend_dir` | `client` | Frontend directory |
| `backend_dir` | `server` | Backend directory |
| `extensions` | `js,jsx,ts,tsx` | File extensions to scan (comma-separated) |

### `risk-policy-gate.yml`

No inputs. Reads `risk-contract.json` from the calling repo root.

### `auto-review.yml`

Requires `ANTHROPIC_API_KEY` secret.

---

## Pipeline Flow

```
PR opened/updated
  │
  ├─ ci-pipeline or ci-pipeline-python
  │   ├─ install deps (cached)
  │   ├─ lint (parallel)
  │   └─ test (parallel)
  │
  ├─ code-health (scorecard comment, non-blocking)
  │
  ├─ risk-policy-gate (after CI passes)
  │
  └─ auto-review (after CI + risk gate)
```

## Requirements

Each project repo needs:
- `risk-contract.json` — copy the template that matches your stack
- `ANTHROPIC_API_KEY` secret (for auto-review)
- Lint and test scripts set up in your project

Optional but recommended:
- `AGENTS.md` — agent-readable project map
- `CLAUDE.md` — style and convention guide
