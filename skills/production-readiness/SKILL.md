---
name: "ralph-loop-v9:production-readiness"
slug: production-readiness
description: >
  13-category launch gate. Run when all tasks in task_status.md are COMPLETE or
  BLOCKED_ERROR. Verifies code quality, test coverage, constitution compliance,
  security, RBAC enforcement, observability, API standards, database conventions,
  documentation, and deployment readiness. Produces docs/production_readiness.md.
---

# /production-readiness — Forge Launch Gate

Run when triggered by /ralph (all issues COMPLETE) or manually by user.

**Prerequisites (must all pass before this skill runs):**
- All issues in `task_status.md` are COMPLETE or BLOCKED_ERROR
- `/build-gate` (full-project) passed with zero errors — if not, fix before proceeding
- `/lsp-report` run with zero Errors — Warnings acceptable

**Inputs:** CONSTITUTION.md, task_status.md, README.md, ai-context/project-constitution.md, ai-context/coding-standards.md

---

## The 13 Categories

### 1. Code Quality
- [ ] /lsp-report: zero Errors across all files
- [ ] No TODO/FIXME/HACK comments in production code
- [ ] No hardcoded secrets, API keys, or credentials
- [ ] No commented-out code blocks > 5 lines
- [ ] No `console.log` / `print` / debug statements in production paths

### 2. Constitution Compliance
- [ ] Run `/constitution check` on entire codebase
- [ ] Zero CRITICAL violations
- [ ] MAJOR violations have documented exceptions with rationale

### 3. Test Coverage
- [ ] Coverage gate met: 80% minimum across all modules
- [ ] Coverage target: 90% or above
- [ ] Critical paths (auth, RBAC, payments, data validation): 100%
- [ ] All UT- IDs from test-plan.md: PASSING
- [ ] All IT- IDs from test-plan.md: PASSING

### 4. Security
- [ ] All endpoints require auth unless explicitly designated public in docs/06_rbac.md
- [ ] RBAC enforcement verified: every role/permission combination in Permission Matrix tested
- [ ] Input validation on all public endpoints (no unvalidated user input reaches DB)
- [ ] Security scan (L6) passing: bandit / OWASP ZAP / Trivy — zero HIGH/CRITICAL findings
- [ ] No secrets in environment (scan with git-secrets or truffleHog)

### 5. RBAC
- [ ] Permission Matrix (docs/06_rbac.md) fully implemented
- [ ] Every role × resource × action combination has a corresponding test
- [ ] Unauthorized access returns 403 (not 401 or 404) in all cases
- [ ] RBAC rules documented and match implementation

### 6. API Standards
- [ ] All endpoints follow consistent patterns (consistent URL structure, status codes, error envelope)
- [ ] Error envelope format consistent across all endpoints
- [ ] HTTP status codes correct per standard (200/201/400/401/403/404/409/422/500)
- [ ] Rate limiting implemented on public endpoints



### 7. Database
- [ ] All migrations run cleanly on a fresh database
- [ ] No N+1 query patterns in production paths
- [ ] All foreign keys indexed
- [ ] Soft-delete implemented consistently across entities
- [ ] Audit trail fields present on required entities

### 8. Observability
- [ ] Structured JSON logging implemented (timestamp, level, service fields)
- [ ] All required log fields present (timestamp, level, service, trace_id)
- [ ] Health endpoint `/health` returns 200
- [ ] Readiness endpoint `/ready` returns 200 when fully initialized
- [ ] Key business metrics instrumented

### 9. Documentation
- [ ] README.md updated with: what it does, how to run locally, how to run tests
- [ ] All public API endpoints documented (inline or OpenAPI spec)
- [ ] CONSTITUTION.md amendment log accurate
- [ ] docs/06_rbac.md matches implementation

### 10. Infrastructure
- [ ] All Category B dependencies (from docs/05_dependencies.md) confirmed or explicitly deferred with documented plan
- [ ] Environment variables documented (name, purpose, which environments)
- [ ] CI pipeline passing on main branch
- [ ] docker-compose up works locally

### 11. E2E Coverage
- [ ] All E2E- IDs from test-plan.md: PASSING against staging
- [ ] Smoke suite (L7) passing against staging
- [ ] No known regression from E2E suite

### 12. Deployment Readiness
- [ ] Deployment process documented in README.md or docs/
- [ ] Rollback procedure documented and tested
- [ ] Staging environment matches production configuration
- [ ] Database migration rollback scripts exist

### 13. Outstanding Issues
- [ ] Zero BLOCKED_ERROR tasks without documented resolution plan
- [ ] Zero open `needs-human` labels on GitHub without triage
- [ ] Zero open HIGH/CRITICAL bugs from Ralph-test or Ralph-e2e

---

## `docs/production_readiness.md`

```markdown
# Production Readiness Report — [Project Name]
Generated: [timestamp]
Status: READY | NOT READY

## Summary
[N]/13 categories PASSED

## Category Results
| # | Category | Status | Notes |
[one row per category]

## Blockers (must fix before launch)
[list]

## Non-blockers (fix post-launch)
[list]
```

---

## Launch Decision

If all 14 categories pass:
```
✅ PRODUCTION READY
All 13 categories passed.
docs/production_readiness.md written.
```

If any CRITICAL category fails:
```
❌ NOT PRODUCTION READY
[N] blockers must be resolved.
See docs/production_readiness.md for details.
```
