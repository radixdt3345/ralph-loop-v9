---
name: "ralph-loop-v9:qmd"
slug: qmd
description: >
  Natural language query across all Forge project documentation. Returns cited
  answers with exact file+section references. Read-only — never modifies files.
  Called internally by /ralph and /production-readiness for targeted lookups.
  Available standalone as /qmd [question].
---

# /qmd — Query Markdown Docs

Read-only. Never modifies files.

**Knowledge base:**
- `CONSTITUTION.md` + `CLAUDE.md` + `task_status.md` + `WORK_PLAN.md`
- `docs/00_discovery.md`, `docs/prd.md`, `docs/02_technical.md`, `docs/06_rbac.md`
- `docs/features/*.md`
- `docs/test-plan.md`

- `docs/lsp/*.md`
- `docs/production_readiness.md` (if exists)
- `ai-context/*.md` and `ai-context/*.json`

---

## Usage

`/qmd [question in plain English]`

Examples:
- `/qmd why is T-042 blocked?`
- `/qmd what are the RBAC rules for the payments domain?`
- `/qmd which tasks depend on AUTH-DB-001?`
- `/qmd what is the integration layer definition for the billing domain?`
- `/qmd which FRs are not yet covered by any task?`
- `/qmd what are the coverage requirements for the auth module?`

---

## Response Format

```
Answer: [direct answer in plain English]

Sources:
- [file path] → [section heading]: [relevant quote or reference]
- [file path] → [section heading]: [relevant quote or reference]
```

If the answer spans multiple files: cite each source separately.
If the question cannot be answered from the knowledge base: say so explicitly.

---

## Internal Use by Other Skills

When /ralph needs to look up:
- RBAC rules for a task → `/qmd RBAC permissions for [domain] [resource]`
- Architecture constraint for a file → `/qmd service boundary rule for [service]`
- Test IDs for a feature → `/qmd test IDs owned by [feature ID]`

When /production-readiness checks:
- `/qmd which issues are BLOCKED_ERROR?`
- `/qmd what are the critical path modules requiring 100% coverage?`
