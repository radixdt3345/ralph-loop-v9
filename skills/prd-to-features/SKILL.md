---
name: "ralph-loop-v9:prd-to-features"
description: >
  Called by /start (Step 5). Do not invoke directly — type /start to begin.
  Decomposes docs/prd.md into Feature files (docs/features/F-XX-slug.md) with User
  Stories, acceptance criteria, waves, and dependencies. Consumed by /feature-to-issues.
---

# /prd-to-features — Feature Decomposition

**Inputs:**
- `docs/prd.md` ✓ (the only required input)

**Outputs:**
- `docs/features/feature-summary.md`
- `docs/features/F-XX-[slug].md` (one per feature)

Run immediately. No preamble.

---

## Step 1 — Parse PRD

Extract from `docs/prd.md`:
- All FR- IDs and their priority (MUST / SHOULD / COULD)
- All AC- IDs mapped to their parent FR-
- All User Roles mentioned
- Any explicit PT- Playwright / E2E scenarios (if present — optional)

If the PRD has no explicit FR- IDs, derive them from the requirements text and number
them FR-1, FR-2, … in the order they appear.

---

## Step 2 — Group into Features

A Feature is a coherent unit of user-facing functionality that:
- Can be named in one short phrase (e.g. "User Authentication", "Invoice Management")
- Maps to one or more FRs
- Can be built and tested independently of other features
- Has clear entry and exit criteria

Assign each feature a unique ID: `F-01`, `F-02`, … (zero-padded).
Slug: kebab-case (e.g. `F-01-user-authentication`).

If the PRD is small (< 5 FRs), it may map to a single feature. That is fine.

---

## Step 3 — Write Feature Files

For each feature, write `docs/features/F-XX-[slug].md`:

```markdown
# F-XX — [Feature Name]

## Purpose
[One paragraph — what this feature does and why it matters to the user]

## User Stories
### US-XX.1: [Story Name]
As a [role], I want to [action] so that [outcome].

**Acceptance Criteria:**
- AC-[n]: [binary, testable condition — passes or fails, no ambiguity]
- AC-[n]: ...

### US-XX.2: [Story Name] (if needed)
...

## Functional Requirements Covered
| FR-ID | Requirement | Priority |
|-------|-------------|----------|
| FR-n  | [text]      | MUST     |

## Roles Involved
[List roles that interact with this feature, from the PRD]

## Execution Wave
Wave [1 / 2 / 3] — [brief rationale]
- Wave 1: foundation / auth / core data model
- Wave 2: primary user-facing workflows
- Wave 3: enhancements, reporting, nice-to-haves

## Dependencies
Depends on: [F-XX, or NONE]
Blocks: [F-XX, or NONE]
```

---

## Step 4 — Write `docs/features/feature-summary.md`

```markdown
# Feature Summary — [Project Name]

## Feature Index
| ID   | Name | Wave | Priority | Depends On |
|------|------|------|----------|------------|
| F-01 | ...  | 1    | MUST     | NONE       |

## Execution Waves
### Wave 1 — Foundation
[Feature IDs + names]

### Wave 2 — Core
[Feature IDs + names]

### Wave 3 — Enhancement
[Feature IDs + names]

## FR Coverage
| FR-ID | Feature | Priority |
|-------|---------|----------|
[every FR-n from the PRD mapped to its F-XX]

## AC Coverage
| AC-ID | User Story | Feature |
|-------|-----------|---------|
[every AC-n mapped to its US and F-XX]
```

---

## ⛔ HIL — Feature Review

After writing all files, present:

```
⛔ FEATURE REVIEW
[N] features across [N] waves

| ID   | Name | Wave | FRs covered | ACs covered |
[one row per feature]

Flagged for discussion:
[List any feature where scope felt ambiguous or a design choice was made]

Type YES to confirm and proceed to /feature-to-issues.
Type REVISE F-XX [change] to update a feature before continuing.
```
