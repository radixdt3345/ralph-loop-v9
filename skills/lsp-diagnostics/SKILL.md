---
name: "ralph-loop-v9:lsp"
description: >
  Two-tier compile and build gate. Tier 1: runs per PR by ralph-impl — LSP diagnostics
  + tsc --noEmit on changed files, zero errors required before merge. Tier 2: runs
  full-project build gate after ALL impl issues for a feature close, before ralph-test
  starts — catches cross-issue wiring errors (missing routes, broken imports, unconnected
  navigation) that per-PR checks cannot see. Zero build errors required before E2E.
  Also available standalone: /lsp-check, /lsp-cross-check, /lsp-report, /build-gate.
---

# /lsp — Compile & Build Gate

**Zero Errors policy (both tiers)**: No PR merges with LSP Errors. No E2E runs with
build errors. Warnings are permitted but must be noted in the PR or build report.

---

## Tier 1 — Per-PR Gate (runs inside every ralph-impl PR)

Run after implementation, before opening PR. Catches errors in the files changed
by this one issue.

### Step 1a — LSP Diagnostics on changed files

Auto-detect language from `ai-context/coding-standards.md`:

| Language | LSP Client |
|----------|-----------|
| TypeScript / JavaScript | typescript-language-server |
| Python | pyright (strict mode) |
| Go | gopls |
| Rust | rust-analyzer |
| C# / .NET | csharp-ls / OmniSharp |
| Java | jdtls |
| Ruby | ruby-lsp |
| PHP | intelephense |

```bash
# Run LSP on files changed in this PR only
git diff --name-only origin/[BASE_BRANCH]...HEAD > /tmp/changed-files.txt
# Pass changed files to LSP client
```

Output:
```
LSP Check — [ISSUE-ID]
Files checked: [N]

ERRORS (blocking — fix before PR):
  [file]:[line]:[col] — [message]

WARNINGS (non-blocking — note in PR):
  [file]:[line]:[col] — [message]

STATUS: CLEAN | ERRORS FOUND
```

If ERRORS FOUND → fix before creating PR. Do not open PR with LSP errors.

### Step 1b — TypeScript compile on changed files (TypeScript projects only)

```bash
# Backend
cd backend && tsc --noEmit 2>&1 | tee /tmp/tsc-backend.txt
BACKEND_ERRORS=$(grep -c "error TS" /tmp/tsc-backend.txt || true)

# Frontend
cd ../frontend && tsc --noEmit 2>&1 | tee /tmp/tsc-frontend.txt
FRONTEND_ERRORS=$(grep -c "error TS" /tmp/tsc-frontend.txt || true)
```

If any errors → fix before PR. Never suppress with `as any` or `@ts-ignore`.

### Step 1c — Unit tests pass

```bash
# Run owned UT- test IDs for this issue
[test runner from ai-context/testing.md] [test file pattern]
```

Coverage must meet the gate in `ai-context/coding-standards.md`.
If below gate → write additional tests. Do not proceed with failing or uncovered suite.

**All three must be CLEAN before ralph-impl opens a PR.**

---

## Tier 2 — Full-Project Build Gate (runs after ALL impl issues for a feature close)

This is the gate that catches **cross-issue wiring errors** — problems that cannot be
seen in any single PR but appear when all the pieces are assembled:

- A route defined in one issue not registered in the router (defined in another issue)
- A nav link in a UI component pointing to a route that doesn't exist yet
- An import from a module created in a sibling issue (file exists now, but wasn't there during per-PR check)
- A DB migration from one issue not reflected in the ORM schema updated in another
- Type errors that only appear when the full type graph is resolved across all modules

**When to run:** ralph-impl orchestrator runs `/build-gate` centrally after every feature's
impl siblings ALL close (in IMPL-7, before signalling ralph-test). Ralph-test will not start
until `/build-gate` passes for that feature.

### `/build-gate [feature-id]`

```bash
# 1. Pull latest — all merged PRs for this feature are now on BASE_BRANCH
git checkout [BASE_BRANCH] && git pull

# 2. Install / sync dependencies
[install command from ai-context/testing.md — e.g. npm ci / pip install / mvn install]

# 3. Full compile — entire project, not just changed files
# TypeScript:
cd backend  && tsc --noEmit 2>&1 | tee /tmp/build-backend.txt
cd frontend && tsc --noEmit 2>&1 | tee /tmp/build-frontend.txt
# Python:
cd backend && python -m py_compile $(find . -name "*.py" | grep -v __pycache__) 2>&1
# Java/Kotlin:
mvn compile -q 2>&1 | tee /tmp/build.txt
# Go:
go build ./... 2>&1 | tee /tmp/build.txt
# C#:
dotnet build --no-restore 2>&1 | tee /tmp/build.txt

# 4. Import / module resolution check
# TypeScript: already covered by tsc --noEmit above
# Python: python -c "import [main_module]" to catch circular import errors
# Java: mvn compile catches missing imports

# 5. Route registration check (web projects)
# Verify every route referenced in navigation/links exists in the router
# Verify every API endpoint called by frontend exists in the backend router
[See Route Audit below]

# 6. Run full unit test suite (all UT- IDs, not just this feature's)
[test runner] --all 2>&1 | tee /tmp/unit-tests.txt
```

### Route Audit (Step 5 — web projects)

For every frontend component that navigates to a route (via `<Link>`, `useNavigate`,
`router.push`, `href`, etc.):

```bash
# Extract all route strings used in navigation
grep -rn "useNavigate\|<Link\|router\.push\|href=" frontend/src/ \
  | grep -oE '["'"'"'][/][a-zA-Z0-9/_:-]+["'"'"']' | sort -u > /tmp/nav-routes.txt

# Extract all registered routes from router config
grep -rn "path:\|route:\|<Route " frontend/src/router* \
  | grep -oE '["'"'"'][/][a-zA-Z0-9/_:-]+["'"'"']' | sort -u > /tmp/registered-routes.txt

# Routes used in nav but not registered = broken navigation
comm -23 /tmp/nav-routes.txt /tmp/registered-routes.txt
```

Any route in `/tmp/nav-routes.txt` not in `/tmp/registered-routes.txt` is a
**broken navigation link** — it will render a blank screen or 404 at E2E time.
Fix before signalling ralph-test.

Similarly for API calls:
```bash
# Extract API endpoint paths called by frontend
grep -rn "axios\.\|fetch(\|api\." frontend/src/ \
  | grep -oE '["'"'"']/api/[a-zA-Z0-9/_:-]+["'"'"']' | sort -u > /tmp/api-calls.txt

# Extract registered backend routes
grep -rn "router\.\(get\|post\|put\|patch\|delete\)\|@Get\|@Post\|@Put" backend/src/ \
  | grep -oE '["'"'"']/api/[a-zA-Z0-9/_:-]+["'"'"']' | sort -u > /tmp/api-routes.txt

# API calls with no matching backend route = runtime 404
comm -23 /tmp/api-calls.txt /tmp/api-routes.txt
```

### Build Gate Output

```
/build-gate — [FEATURE-ID]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Full compile:
  Backend:  [CLEAN | N ERRORS]
  Frontend: [CLEAN | N ERRORS]

Import/module resolution: [CLEAN | N ERRORS]

Route audit:
  Nav routes checked: [N]
  Unregistered nav routes: [N] [CLEAN | BROKEN]
    [/path/broken] — used in [file:line], not in router

API endpoint audit:
  Frontend API calls checked: [N]
  Unmatched backend routes: [N] [CLEAN | BROKEN]
    [/api/path] — called in [file:line], not registered in backend

Unit test suite: [N passed / N failed]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RESULT: ✅ PASSED — ralph-test may proceed
        ❌ FAILED — [N] issues must be fixed before ralph-test starts
```

### Build Gate Failure Handling

**If build gate fails:**

1. Open a new GitHub issue titled:
   `[BUILD-GATE][F-XX] [brief description of failure]`
   Body: full build gate output, which files are broken, which PR introduced the issue
   Labels: `build-gate-failure`, `needs-fix`

2. Block ralph-test from starting on this feature — post a comment on the TEST issue:
   > ⛔ Build gate failed for [FEATURE]. Fix [BUILD-GATE issue #N] before integration
   > tests can run. Re-run `/build-gate [feature-id]` after fixes are merged.

3. Do NOT open a new issue per error — one build-gate issue per feature per run.

4. Ralph-impl should fix these by:
   - Checking out BASE_BRANCH
   - Creating a fix branch: `fix/build-gate-[feature-id]-[slug]`
   - Fixing the wiring errors (register missing routes, fix imports, align types)
   - Running `/build-gate` again to confirm clean
   - Merging the fix PR
   - Re-running `/build-gate` one final time to confirm before signalling ralph-test

---

## Standalone Commands

### `/lsp-check [path]`

Run Tier 1 LSP on a specific file or directory on demand.

```
LSP Check: [path]
ERRORS: [N] | WARNINGS: [N]
[file:line:col — message]
STATUS: CLEAN | ERRORS FOUND
```

### `/lsp-cross-check`

Run full TypeScript compile across backend + frontend + shared:

```bash
cd backend  && tsc --noEmit
cd frontend && tsc --noEmit
cd shared   && tsc --noEmit   # if shared types dir exists
```

```
Cross-Service TypeScript Check
Backend:  [N errors] [CLEAN | ERRORS]
Frontend: [N errors] [CLEAN | ERRORS]
Shared:   [N errors] [CLEAN | ERRORS]
```

### `/lsp-report`

Full-repo LSP sweep. Output: `docs/lsp/lsp-report.md`.

```markdown
# LSP Report — [Project Name]
Generated: [timestamp]

## Summary
| Layer | Files | Errors | Warnings |
|-------|-------|--------|----------|
| Backend | [N] | [N] | [N] |
| Frontend | [N] | [N] | [N] |

## Errors (must fix)
[file:line:col — message]

## Warnings (review)
[file:line:col — message]
```

---

## Common Errors and Fixes

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| `Cannot find module '...'` | Import path wrong or file not yet created | Check file was created in a sibling PR and is on BASE_BRANCH |
| `Property 'X' does not exist` | Type mismatch between two PRs | Align types — check which PR owns the type definition |
| `No overload matches this call` | API signature changed in one PR, caller not updated | Update call site to match current signature |
| Route used but not registered | Nav link created before router updated | Add route to router config; check SCR-NNN mockup for correct path |
| `/api/X` called but not registered | Frontend API module created before backend route | Confirm backend route PR is merged; check route string matches exactly |
| `Module has no exported member` | Export added in one PR, import in another before merge | Pull BASE_BRANCH and confirm all PRs are merged |
| Blank screen on navigation | Route not registered, or component not mounted in layout | Run route audit — `/build-gate` route check section |
