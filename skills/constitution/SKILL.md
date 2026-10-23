---
name: "ralph-loop-v9:constitution"
description: >
  Called by /start (Step 4). Do not invoke directly — type /start to begin.
  Generates CONSTITUTION.md plus 5 ai-context bridge files read by all agents:
  project-constitution.md, coding-standards.md, testing.md, ralph-agent-spec.md,
  and database-guidelines.md. Asks 8 quick questions inline — no external files needed
  beyond docs/prd.md. Exception: /constitution amend and /constitution check are
  standalone commands.
---

# /constitution — Project Engineering Constitution

Run immediately. No preamble. Three modes: CREATE (default), AMEND, CHECK.

---

## CREATE Mode

**Inputs:**
- `docs/prd.md` ✓ (the only required file)
- User answers to 8 quick questions (asked inline — no separate file needed)

**Outputs:**
- `CONSTITUTION.md` — canonical, locked after ratification
- `ai-context/project-constitution.md` — quick-reference read by all agents per task
- `ai-context/coding-standards.md` — code rules read by ralph-impl per task
- `ai-context/testing.md` — test framework + environment config read by all three agents
- `ai-context/ralph-agent-spec.md` — branch strategy + concurrency config read by all three agents
- `ai-context/database-guidelines.md` — DB naming + migration rules read by ralph-impl

---

### Step 1 — Ask 8 Questions

Ask these one at a time. Use the PRD to pre-fill sensible defaults where possible.
The user can accept a default or override it.

```
1. Tech stack
   What languages / frameworks will this project use?
   (e.g. "TypeScript + React + Node.js + PostgreSQL"
         "Python + FastAPI + React + MySQL"
         "Java + Spring Boot + Angular + PostgreSQL")

2. Code formatter + linter
   What formatter and linter should be enforced?
   (e.g. "Prettier + ESLint" / "Black + Ruff" / "Spotless + Checkstyle")

3. Unit test framework
   What unit test framework will be used for backend? And frontend (if applicable)?
   (e.g. "Vitest + @testing-library/react for frontend, Jest for backend"
         "pytest" / "JUnit 5" / "xUnit")

4. Integration + E2E test tools
   What tools for integration tests and E2E browser tests?
   (e.g. "Supertest for integration, Playwright for E2E"
         "pytest + httpx for integration, Playwright for E2E"
         "REST Assured for integration, Cypress for E2E")

5. Test environment
   How is the test/integration environment run locally?
   (e.g. "docker-compose up" / "npm run dev + separate test DB"
         "embedded H2 for integration tests" / "pytest fixtures with SQLite")
   Include the health-check URL pattern if known (e.g. "http://localhost:3000/health")

6. Auth mechanism
   How will authentication work?
   (e.g. "JWT in Authorization header" / "Session cookies" / "OAuth2 via provider")

7. Git branching strategy
   What is the branch and PR strategy?
   (e.g. "trunk-based: feature branches → main directly"
         "tiered: feature branches → dev → main"
         "gitflow: feature → develop → release → main")
   Also: what is the PR target branch? (e.g. "main" / "dev")

8. Any hard constraints or non-negotiables?
   (e.g. "GDPR compliance required" / "No third-party analytics" / "Must run offline"
         Leave blank if none)

9. Does this project have a frontend? (YES / NO)
   If YES: what frontend stack?
   (e.g. "React 18 + Vite + MUI v5 + Redux Toolkit"
         "Next.js 14 + Tailwind CSS"
         "Vue 3 + Vuetify"
         "Angular 17 + PrimeNG")
   This answer is written to ralph-agent-spec.md and enforced by /ralph Phase A and /feature-to-issues.
```

---

### Step 2 — Draft Constitution

Write `CONSTITUTION.md` with these 5 articles:

```markdown
# CONSTITUTION.md — [Project Name]
Ratified: [date]
Version: 1.0

---

## Article I — Core Principles

[3–5 immutable rules derived from PRD constraints + user answers. Examples:]
- No secrets or credentials in source code — environment variables only
- Every authenticated endpoint enforces auth before returning data
- No direct DB access across service boundaries — API only
- [Any hard constraint the user gave in question 8]

These principles cannot be overridden by any task, PR, or agent instruction.

---

## Article II — Code Quality Standards

**Stack:** [from question 1]

**Formatter:** [from question 2] — enforced in CI, no exceptions
**Linter:** [from question 2] — zero warnings policy on new code

**Naming conventions:**
| Scope | Convention | Example |
|-------|-----------|---------| 
| Files (frontend components) | PascalCase | UserProfile.tsx |
| Files (backend modules) | camelCase or snake_case | userService.ts |
| Functions / methods | camelCase | getUserById() |
| Variables | camelCase | isLoading |
| Constants | UPPER_SNAKE_CASE | MAX_RETRIES |
| DB tables | snake_case, plural | user_profiles |
| DB columns | snake_case | created_at |
| CSS classes | kebab-case | primary-btn |

**Forbidden patterns (agents must never produce these):**
- `any` type in TypeScript (or equivalent unsafe cast in other languages)
- `console.log` / `print` statements in production code paths
- Hardcoded credentials, URLs, or environment-specific values in source
- `TODO` comments committed to main branch
- Commented-out code blocks

**PR requirements:**
- Every PR must have a description explaining what changed and why
- No self-merge without CI passing
- No `--force` push to main or staging branches

---

## Article III — Testing Policy

**Unit test framework (backend):** [from question 3]
**Unit test framework (frontend):** [from question 3]
**Integration test tool:** [from question 4]
**E2E test tool:** [from question 4]
**Test environment:** [from question 5]

**Coverage gates (enforced in CI — PRs below gate are blocked):**
| Scope | Gate | Target |
|-------|------|--------|
| All code | 80% lines | 90% |
| Auth / security paths | 100% | 100% |
| Payment / data-write paths | 100% | 100% |

**Test rules:**
- Unit tests co-located with code or in a `tests/` sibling directory
- No `time.sleep` / `setTimeout` in tests — use mocks
- Tests must be order-independent (each sets up and tears down its own state)
- E2E tests trigger only via `workflow_dispatch` — never on push/pull_request
- Mocked return values must match the real API response shape exactly

---

## Article IV — Security Baseline

**Auth mechanism:** [from question 6]
**Enforcement point:** Middleware / guard layer — never inside individual route handlers

**Rules:**
- All user input validated and sanitized before use
- Secrets in environment variables only — never in code, config files, or logs
- Sensitive data (passwords, tokens) never logged at any level
- Error responses never expose internal stack traces or DB details to clients
- [GDPR / compliance rules from question 8, if any]

---

## Article V — Amendment Process

Amendments only via `/constitution amend [article] [change]`.
Every amendment logged in the Appendix below with date and rationale.
`CONSTITUTION.md` may not be edited directly by any agent.

---

## Appendix — Amendment Log
[empty at ratification]
```

---

### Step 3 — HIL Ratification

Present the full draft and halt:

```
⛔ CONSTITUTION RATIFICATION

Review all 5 articles above.
Once ratified, changes require /constitution amend.

Type YES to ratify and generate all bridge files.
Type REVISE [article] [change] to update before ratifying.
```

---

### Step 4 — Write Bridge Files (after ratification)

Write all 5 files. Every file must be complete — no placeholders.

---

#### `ai-context/project-constitution.md`

Read by all three agents (ralph-impl, ralph-test, ralph-e2e) at startup.
Must be concise — agents read it in full on every task.

```markdown
# Project Constitution — Quick Reference
Source: CONSTITUTION.md v[N] (ratified [date])

## Immutable Principles
[bullet list from Article I — 3–5 items max]

## Stack
[one line: languages, frameworks, DB]

## Formatter / Linter
[one line: tools]

## Naming (quick ref)
| Scope | Convention |
[condensed table from Article II]

## Forbidden (agents must never produce)
[bullet list from Article II forbidden patterns]

## Test Framework
Backend unit: [framework] | Frontend unit: [framework]
Integration: [tool] | E2E: [tool]
Coverage gate: 80% lines | Auth/payment paths: 100%

## Auth Enforcement
[one line: mechanism + enforcement point]

## Security Rules
[3–4 bullet points from Article IV]
```

---

#### `ai-context/coding-standards.md`

Read by ralph-impl when generating or reviewing code.

```markdown
# Coding Standards — [Project Name]

## Language & Tooling
| Item | Value |
|------|-------|
| Stack | [languages + frameworks] |
| Formatter | [tool] |
| Linter | [tool] |
| Backend unit test | [framework] |
| Frontend unit test | [framework] |
| Integration test | [tool] |
| E2E test | [tool] |

## Naming Conventions
[Full table from Article II]

## Forbidden Patterns
[Full list — agents treat these as hard errors]
- [ ] No `any` type (TypeScript) / unsafe cast
- [ ] No `console.log` / `print` in production paths
- [ ] No hardcoded credentials, URLs, or env-specific values
- [ ] No `TODO` comments on main branch
- [ ] No commented-out code blocks

## Test Rules
- Backend unit framework: [name] | Coverage gate: 80% lines (CI blocking)
- Frontend unit framework: [name]
- Mock shapes must match real API response shapes exactly
- No time.sleep / setTimeout in tests
- E2E tests via workflow_dispatch only — never wired to push/pull_request

## PR Checklist (ralph-impl runs this before every PR)
- [ ] Formatter ran — no diff
- [ ] Linter passes — zero warnings on changed files
- [ ] All owned test IDs written and passing
- [ ] Coverage gate met
- [ ] No forbidden patterns in changed files
- [ ] PR description explains what changed and why
```

---

#### `ai-context/testing.md`

Read by all three agents (ralph-impl, ralph-test, ralph-e2e).
This is the file agents use to know HOW to run tests and WHERE to put test files.

```markdown
# Testing Configuration — [Project Name]
Source: CONSTITUTION.md Article III (ratified [date])

## Unit Tests — Backend
Framework: [from question 3]
File location: [derive from stack — e.g. tests/ alongside src/, or __tests__/ in each module]
File naming: [e.g. *.test.ts / test_*.py / *Tests.java]
Run command: [e.g. "npm test" / "pytest" / "mvn test"]
Coverage gate: 80% lines | Target: 90%
Coverage run command: [e.g. "npm run test:coverage" / "pytest --cov"]

## Unit Tests — Frontend
Framework: [from question 3 — if frontend exists, else omit section]
File location: [e.g. src/tests/ or co-located __tests__/]
File naming: [e.g. *.test.tsx]
Run command: [e.g. "npm run test:frontend"]

## Integration Tests
Tool: [from question 4]
File location: [e.g. tests/integration/ / src/**/*.integration.test.ts]
File naming: [e.g. *.integration.test.ts / test_*_integration.py]
Run command: [e.g. "npm run test:integration" / "pytest tests/integration/"]
Environment: [from question 5 — e.g. "docker-compose up -d before running"]
Health check URL: [e.g. "http://localhost:3000/health" — or "N/A"]
Test must be independent: yes — no shared mutable state between IT- tests
Real infrastructure: yes — no mocking at integration layer unless stated below
[Exception: [any stated mock exception, or "none"]]

## E2E Tests
Tool: [from question 4 — Playwright or Cypress]
Config file: [e.g. playwright.config.ts / cypress.config.ts]
File location: [e.g. e2e/ / tests/e2e/]
File naming: [e.g. *.e2e.ts / *.cy.ts]
Run command: [e.g. "npx playwright test" / "npx cypress run"]
Staging base URL: [to be filled in after /platform-setup — placeholder: TBD]
Browser targets: [e.g. "chromium, firefox" / "electron (Cypress default)"]

## E2E Test Trigger Model
⚠️ E2E tests MUST NOT be wired to CI push/pull_request triggers.
Trigger: workflow_dispatch only (manually invoked by ralph-e2e or a human).
Rationale: E2E tests run against staging — staging is not available in PR CI.
CI pipeline must NOT include an E2E step in its on: push or on: pull_request jobs.

## Test Environment Setup
[From question 5 — step-by-step:]
1. [e.g. "docker-compose up -d" / "Start the dev server: npm run dev"]
2. [e.g. "Wait for health check: curl http://localhost:3000/health"]
3. [e.g. "Seed test data: npm run db:seed:test" — or "N/A"]
Teardown: [e.g. "docker-compose down" / "N/A"]
```

---

#### `ai-context/ralph-agent-spec.md`

Read by all three agents at startup to determine branch strategy, PR target,
concurrency caps, and failure behaviour. Derive from questions 1, 7, and stack.

```markdown
# Ralph Agent Specification — [Project Name]
Source: CONSTITUTION.md (ratified [date])

## Branch Strategy
Model: [trunk-based / tiered / gitflow — from question 7]
PR target: [exact branch name — e.g. "main" / "dev" — from question 7]
Feature branch prefix: feat/
Test branch prefix: test/
E2E branch prefix: e2e/
Branch naming: [prefix]/[ISSUE-ID]-[slug]
  Examples:
    feat/F01-001-user-authentication
    test/F01-002-auth-integration-tests
    e2e/F01-003-auth-e2e-tests

## Merge Strategy
Merge type: squash merge
CI gate: must pass before merge
Force push: never allowed on [PR target branch]

## Parallelization Model (ralph-impl)
Max parallel domain agents (Tier 1): 4
Max parallel issue workers per stage (Tier 2): 3
Worktree path convention: ../[repo-slug]-wt-[domain-lowercase]
  Example: ../my-app-wt-auth

## Max Turns
ralph-impl (per issue): 60
ralph-test (per TEST issue): 40
ralph-e2e (per E2E issue): 50

## Failure Labels
| Situation | Label |
|-----------|-------|
| Max turns exceeded | needs-human |
| CI failing, unresolvable | needs-human |
| Test environment unreachable | env-issue |
| Staging unreachable (e2e) | env-issue |
| Merge conflict, unresolvable | needs-human |
| LSP errors after 3 attempts | lsp-blocked |

## Domain Map
[Derive from features in docs/features/feature-summary.md — one domain per major bounded context.
If the project is small / single-domain, write "MAIN" as the single domain.]

| Domain Code | Description | Owned feature(s) |
|-------------|-------------|-----------------|
| [CODE] | [what this domain covers] | [F-XX, F-XX] |

## Frontend Declaration
has_frontend: [true / false — from question 9]
frontend_stack: [framework + libraries — from question 9, or "N/A" if has_frontend is false]

⚠️ If has_frontend is true, /ralph Phase A will halt if ai-context/ui-design.md is absent.
   Run /design-ui BEFORE /ralph when has_frontend is true.
   /feature-to-issues will generate UI-layer issues for every feature only when has_frontend is true.

## Agent Invocation
ralph-impl: claude --agent ralph-loop-v9:ralph-impl  (or: /ralph in Claude Code)
ralph-test:  claude --agent ralph-loop-v9:ralph-test
ralph-e2e:   claude --agent ralph-loop-v9:ralph-e2e
```

---

#### `ai-context/database-guidelines.md`

Read by ralph-impl for every DB-layer issue.
Derive from questions 1 and 8 (stack = DB technology; constraints may add rules).

```markdown
# Database Guidelines — [Project Name]
Source: CONSTITUTION.md Article II (ratified [date])

## Technology
Database: [from question 1 — e.g. PostgreSQL / MySQL / MongoDB / SQLite]
ORM / query builder: [derive from stack — e.g. Prisma / TypeORM / SQLAlchemy / Hibernate]
Migration tool: [derive from stack — e.g. Prisma Migrate / Flyway / Alembic / Liquibase]

## Naming Conventions
| Object | Convention | Example |
|--------|-----------|---------|
| Tables | snake_case, plural | user_profiles |
| Columns | snake_case | created_at |
| Primary keys | id (UUID) | id uuid default gen_random_uuid() |
| Foreign keys | [referenced_table]_id | user_id |
| Indexes | idx_[table]_[column(s)] | idx_users_email |
| Junction tables | [table_a]_[table_b] (alphabetical) | role_users |

## Migration Rules
- Create new migration files only — never modify an existing migration
- Every migration must include a rollback (down migration)
- Migration file naming: [timestamp]_[description].[ext]
  e.g. 20240115_create_users_table.sql
- Run migrations before seeding in any environment
- Never run migrations directly on production — always via CI/CD pipeline

## Data Integrity Rules
- Every table has: id (PK), created_at, updated_at
- Soft delete: use deleted_at timestamp column — never hard delete user-generated data
- All foreign key constraints enforced at DB level (no application-only FK enforcement)
- Unique constraints defined in migration, not only at application layer

## Seeding
- Test seed files live in: [e.g. prisma/seed-test.ts / seeds/test/]
- Production seed files (reference data only) live in: [e.g. prisma/seed.ts / seeds/prod/]
- Never use production seed data in test environments
- Seeds must be idempotent (safe to run multiple times)

## Connection
- Connection string in environment variable only — never hardcoded
- Environment variable name: DATABASE_URL (or [project-specific name from question 1])
- Connection pooling: enabled in production (pool size: 10 default)
- Separate DB per environment: dev / test / staging / production
```

---

## AMEND Mode

`/constitution amend [article] [change]`

1. Show current article text
2. Show proposed change
3. Ask: `Confirm amendment? YES / NO`
4. On YES: update `CONSTITUTION.md`, regenerate ALL 5 bridge files, append to Amendment Log

When regenerating after an amendment, re-derive only the bridge files affected by the
changed article. Always regenerate `project-constitution.md` regardless.

---

## CHECK Mode

`/constitution check [file or directory]`

Scan the target against Articles II–IV. Report:

```
CONSTITUTION CHECK — [path]
Article II violations: [N]
  [file:line] — [violation description]
Article III violations: [N]
  [file:line] — [violation description]
Article IV violations: [N]
  [file:line] — [violation description]

Severity: ERROR (blocks PR) / WARNING (log and proceed)
```

Article II forbidden patterns = ERROR.
Article III coverage below gate = ERROR.
Article IV security rules = ERROR.
