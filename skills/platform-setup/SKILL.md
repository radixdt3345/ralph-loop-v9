---
name: "ralph-loop-v9:platform-setup"
slug: platform-setup
description: >
  Called by /start (Step 7). Do not invoke directly — type /start to begin.
  Creates the GitHub repository and project board. Issue creation is handled by
  /push-to-pms. Supports GitHub (default), GitLab, and Azure DevOps.
---

# /platform-setup — Forge Repo & Board Setup

**Scope:** Repository creation + project board configuration ONLY.
Issue creation is handled exclusively by /push-to-pms.

**Inputs:** `ai-context/issues.json` ✓, `task_status.md` ✓

---

## Step 1 — Platform Selection

Ask:
```
Which platform for your repository and issue tracking?
  1. GitHub (default)
  2. GitLab
  3. Azure DevOps

Enter 1, 2, or 3:
```

Then collect credentials:
- GitHub: Personal Access Token (scopes: repo, project)
- GitLab: Personal Access Token (scopes: api, write_repository)
- Azure DevOps: Personal Access Token + organization URL

Store credentials reference in `docs/05_dependencies.md` (never the values).

---

## Step 2 — Create Repository

### GitHub
```
Repository name: [from ai-context/project.json slug]
Visibility: private (ask user to confirm if they want public)
Default branch: main
Initialize: yes (with .gitignore for the detected stack)
```

Configure:
- Branch protection on `main`: require PR, require CI, no force push, no direct push
- If tiered branch strategy: also protect `dev`

### GitLab
Create project in specified namespace. Set default branch. Configure branch protection.

### Azure DevOps
Create repository in specified project. Set default branch. Configure branch policies.

---

## Step 3 — Configure CI/CD

### GitHub Actions
Create `.github/workflows/ci.yml`:

```yaml
name: "ralph-loop-v9:platform-setup"
on:
  pull_request:
    branches: [main]  # or [dev, main] if tiered
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup [language]
        # From docs/02_technical.md
      - name: Install dependencies
        run: [install command]
      - name: Lint
        run: [lint command]
      - name: Unit tests + coverage
        run: [test command with coverage]
      - name: Coverage gate
        run: [coverage check >= 80%]
      - name: Integration tests
        run: [integration test command]
```

Do NOT add E2E/Playwright to this workflow — per ai-context/testing.md E2E Trigger Model.

### GitLab CI
Create `.gitlab-ci.yml` with equivalent stages.

### Azure DevOps
Create `azure-pipelines.yml` with equivalent stages.

---

## Step 4 — Create Project Board

### GitHub Projects v2
Create board with columns:
- Backlog
- In Progress
- In Review
- Done

Add milestone per Epic (E-NN → GitHub Milestone).

### GitLab
Create board with same columns. Add milestones per epic.

### Azure DevOps
Create board with Backlog / Active / Resolved / Closed.

---

## Step 5 — Write `docs/platform_map.md`

```markdown
# Platform Map — [Project Name]

## Repository
Platform: [GitHub / GitLab / Azure DevOps]
URL: [repo URL]
Default branch: main
CI workflow: [.github/workflows/ci.yml]

## Project Board
URL: [board URL]

## Milestones
| Epic ID | Milestone | URL |
| E-01 | [name] | [URL] |

## Credentials
GitHub PAT: stored in [env var name — value never logged]
```

---

## Step 6 — Confirm

Print:
```
✅ Repository created: [URL]
✅ CI configured: [workflow file]
✅ Project board: [URL]
✅ [N] milestones created (one per Epic)

Next: /push-to-pms will create [N] issues in GitHub Issues.
```
