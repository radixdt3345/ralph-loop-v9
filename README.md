# ralph-loop-v9 — Forge Multi-Agent Build Orchestrator

Version 7.0.0 — Clean rebuild with mandatory frontend enforcement.

## What's New in v7 (vs v6)

### 1. `has_frontend` Guardrail in `/constitution`
`/constitution` now asks question 9: **"Does this project have a frontend?"**  
The answer is written as `has_frontend: true/false` and `frontend_stack` into `ai-context/ralph-agent-spec.md`.  
All subsequent skills read this flag to know whether to generate and enforce UI work.

### 2. Layer-Split Issues in `/feature-to-issues`
Issues are no longer generated per User Story. Every feature now generates **one issue per implementation layer**:

| Layer | Issue ID | Who implements |
|-------|----------|----------------|
| DB | F01-DB-001 | ralph-impl |
| API | F01-API-001 | ralph-impl |
| UI | F01-UI-001 | ralph-impl (only if has_frontend: true) |
| INT | F01-INT-001 | ralph-impl |
| TEST | F01-TEST-001 | ralph-test |
| E2E | F01-E2E-001 | ralph-e2e |

UI issues reference `SCR-NNN` screen IDs from `/design-ui`. If `ui-design.md` is absent, UI issues are NOT generated — the HIL gate halts and instructs the user to run `/design-ui` first.

### 3. `/design-ui` Mandatory Prerequisite in `/ralph` Phase A
`/ralph` Phase A1.5 now checks:
- If `has_frontend: true` and `ai-context/ui-design.md` is **missing** → **HALT**, instruct user to run `/design-ui` first.
- If `ui-design.md` exists but `issues.json` has **zero UI-layer issues** → **HALT**, instruct user to re-run `/feature-to-issues`.

### 4. React DoD Hard Rule in `ralph-impl`
For every UI-layer issue, ralph-impl now:
- **Must** create at least one `.tsx` file in `frontend/src/` — or it does NOT open a PR.
- Enforces `response.data.data` pattern (no bare arrays or `.items`/`.result`).
- Requires `data-testid` on every interactive element.
- Requires frontend unit test file (`frontend/src/tests/[ComponentName].test.tsx`) committed in the same PR.
- Labels `ui-impl-missing` and marks `BLOCKED_ERROR` if any of the above fail.

## Workflow Order

```
/start
  └── /write-prd          (Step 1) — Draft PRD
  └── /prd-to-features    (Step 3) — Feature decomposition
  └── /constitution       (Step 4) — Engineering constitution + has_frontend flag ✨
  └── /design-ui          (Step 5) — Screen wireframes + SCR-NNN IDs [REQUIRED if has_frontend]
  └── /write-test-plan    (Step 6) — Test plan
  └── /feature-to-issues  (Step 7) — Layer-split issue manifest ✨
  └── /platform-setup     (Step 8) — GitHub repo setup
  └── /push-to-pms        (Step 9) — Push issues to GitHub
  └── /ralph              (Step 10) — Multi-agent build orchestrator ✨
        Phase A — Init + Frontend Gate ✨
        Phase B — ralph-impl (DB → API → UI → INT per domain)
        Phase C — ralph-test (integration tests)
        Phase D — ralph-e2e (E2E tests)
        Phase E — /production-readiness
```

## Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| constitution | `/ralph-loop-v9:constitution` | Engineering constitution + frontend declaration |
| design-ui | `/ralph-loop-v9:design-ui` | Screen wireframes, SCR-NNN IDs, mockups |
| feature-to-issues | `/ralph-loop-v9:feature-to-issues` | Layer-split issue manifest |
| ralph | `/ralph-loop-v9:ralph` | Build orchestrator |
| prd-to-features | `/ralph-loop-v9:prd-to-features` | Feature decomposition |
| push-to-pms | `/ralph-loop-v9:push-to-pms` | Issue publisher |
| write-test-plan | `/ralph-loop-v9:write-test-plan` | Test plan |
| platform-setup | `/ralph-loop-v9:platform-setup` | GitHub repo setup |
| write-prd | `/ralph-loop-v9:write-prd` | PRD authoring |
| production-readiness | `/ralph-loop-v9:production-readiness` | Final readiness check |
| start | `/ralph-loop-v9:start` | Workflow entry point |
| status | `/ralph-loop-v9:status` | Build status overview |
| grill | `/ralph-loop-v9:grill` | Requirements review |
| qmd | `/ralph-loop-v9:qmd` | Document indexing |

## Agents

| Agent | Purpose |
|-------|---------|
| `ralph-loop-v9:ralph-impl` | Implementation (DB/API/UI/INT) — enforces React DoD |
| `ralph-loop-v9:ralph-test` | Integration tests (TEST issues) |
| `ralph-loop-v9:ralph-e2e` | E2E tests (E2E issues) |
