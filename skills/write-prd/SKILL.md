---
name: "ralph-loop-v9:write-prd"
slug: write-prd
description: >
  Called by /start (Step 3). Do not invoke directly — type /start to begin.
  Generates docs/prd.md with all requirements tagged FR-n, AC-n, PT-n.
  Includes a HIL checkpoint. After user confirms, /start continues to /constitution.
---

# /write-prd — Forge PRD Generator

**Input:** Clarity Summary from /grill (in context) + `docs/00_discovery.md`

**Output:** `docs/prd.md` + `ai-context/project.json`

Run immediately. No preamble.

---

## Step 1 — Load Context

Read:
- Clarity Summary from /grill (in context — must-have features, constraints, risks)
- `docs/00_discovery.md` — full discovery record
- `ai-context/tech-stack.md` — if it exists (may not exist yet; if missing, proceed)

---

## Step 2 — Generate `docs/prd.md`

Write the PRD with exactly the following 16 sections. Every requirement MUST have a
unique ID. No requirement may be described without an FR- ID.

### Section 1 — Executive Summary
- Product name and one-paragraph description
- Problem being solved
- Target users (from Role Registry in discovery)
- Success metrics (what "done" looks like)

### Section 2 — Scope
- In scope: explicit list
- Out of scope: explicit list (anything mentioned but deferred)
- Future phases: brief note on what's excluded now but planned later

### Section 3 — User Roles
| Role | Description | Primary Goals |
Copy from discovery Role Registry. Add "Guest" / "Unauthenticated" if applicable.

### Section 4 — Functional Requirements

Format every requirement as:

```
FR-[N]: [One sentence describing what the system must do]
Priority: MUST / SHOULD / COULD
Roles: [which roles this applies to]
```

Number from FR-1 upward. Never skip a number. Cover every feature from discovery.

Group by domain for readability.

### Section 5 — Acceptance Criteria

For every FR- that has testable behaviour, write AC-n entries:

```
AC-[N] (FR-[ref]): [Binary condition — either passes or fails. No ambiguity.]
```

Number from AC-1 upward. Every MUST priority FR must have at least one AC.
An AC must be binary: it either passes or fails. Never write "should work well" —
write "returns HTTP 200 with user object containing id, email, role".

### Section 6 — User Stories

```
US-[N] (FR-[refs]):
As a [role], I want to [action] so that [outcome].

Acceptance:
- AC-[n]: [copy from Section 5]
```

### Section 7 — Non-Functional Requirements

```
NFR-[N]: [Requirement]
Category: Performance / Security / Availability / Scalability / Compliance
Measurable target: [specific number or threshold]
```

From Round 5 of discovery. Every NFR must have a measurable target.

### Section 8 — Data Model (Conceptual)

For each entity:
```
Entity: [Name]
Fields: [list of fields with type and constraint]
Relationships: [one-to-many, etc.]
RBAC: [which roles can read/write/delete]
```

### Section 9 — API Surface (Conceptual)

For each major endpoint or operation:
```
[METHOD] /path
Purpose: [what it does]
Auth: [required role]
Request: [key fields]
Response: [key fields]
Errors: [key error cases]
```

### Section 10 — Integration Points

List every external service or system:
```
Integration: [Service name]
Purpose: [what it's used for]
Direction: inbound / outbound / bidirectional
Auth method: [API key / OAuth / webhook / etc.]
Failure handling: [what happens if this service is down]
```

### Section 11 — Security Requirements

```
SEC-[N]: [Security requirement]
Category: Auth / Data / Network / Compliance / Secrets
Implementation note: [brief technical direction]
```

### Section 12 — Constraints and Assumptions

List:
- Technical constraints (must use X, cannot use Y)
- Business constraints (budget, timeline, regulation)
- Assumptions (things treated as true but not confirmed)

### Section 13 — Out of Scope (Detailed)

For each out-of-scope item, explain WHY it's excluded and what would change that.

### Section 14 — Open Questions

```
Q-[N]: [Question]
Owner: [who needs to answer this]
Impact if unresolved: [what can't proceed without this]
Status: OPEN / DEFERRED / ANSWERED
```

### Section 15 — E2E Test Scenarios (Playwright)

For each critical user flow, write a Playwright scenario:

```
PT-[N] (FR-[refs], AC-[refs]):
Scenario: [human-readable name]
Actor: [role]
Steps:
  1. [action]
  2. [action]
  ...
Expected: [final state]
```

Number from PT-1 upward. Every MUST priority user journey must have a PT- scenario.

### Section 16 — HITL Review Checkpoint ⛔

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FORGE  HIL 1 — PRD REVIEW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Requirements summary:
- [N] Functional Requirements (FR-1 to FR-N)
- [N] Acceptance Criteria (AC-1 to AC-N)
- [N] Playwright Scenarios (PT-1 to PT-N)
- [N] Open Questions requiring resolution

Review checklist:
□ Every MUST feature has at least one FR-
□ Every MUST FR has at least one AC-
□ Every AC is binary (pass/fail)
□ Every critical user journey has a PT-
□ Scope boundary is clear
□ No open questions will block Phase 1

Type YES to confirm the PRD and proceed to /constitution.
Type REVISE [section] to update specific sections.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Do NOT proceed until user confirms.

---

## Step 3 — Write `ai-context/project.json`

```json
{
  "name": "[project name]",
  "slug": "[url-safe slug]",
  "created_at": "[ISO date]",
  "workflow": "forge-greenfield",
  "artifacts": {
    "prd": "docs/prd.md",
    "discovery": "docs/00_discovery.md",
    "feature_summary": "docs/features/feature-summary.md",
    "issues": "ai-context/issues.json",
    "task_status": "task_status.md"
  },
  "counts": {
    "fr": [N],
    "ac": [N],
    "pt": [N]
  }
}
```

---

After writing both files, hand control back to /start for Step 5 (/constitution).
