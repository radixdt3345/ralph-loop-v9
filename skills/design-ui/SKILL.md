---
name: "ralph-loop-v9:design-ui"
description: >
  Called by /start (Step 6). Do not invoke directly — type /start to begin.
  Designs the UI/UX for approved features before issues are created: screens, navigation,
  layout, and visual style derived from User Stories. Runs between /prd-to-features and
  /feature-to-issues. After producing mockups, writes Screen References back into each
  F-XX-*.md feature file so /feature-to-issues has a single authoritative source per feature.
---

# /design-ui — UI Design Elicitation and Mockup Generator

## Workflow Role

`/design-ui` is **Step 6 of the Forge pipeline** — between `/prd-to-features` and `/feature-to-issues`.

```
FORGE PIPELINE:
  /start → /grill → /write-prd → /constitution → /prd-to-features
    → HITL Feature Review → [/design-ui] → HITL UI Design Review
    → /feature-to-issues → HITL Issue Review → /platform-setup → /push-to-pms → /ralph
```

**Input:** Approved Feature Breakdown (`docs/features/*.md`) + `ai-context/project-constitution.md` / `ai-context/coding-standards.md` + `ai-context/design-system.md` if present.

**Output:** `docs/design/ui-design.md` + `docs/design/mockups/*.html` + `ai-context/ui-design.md` + **Screen References written back into each `docs/features/F-XX-*.md`**.

---

## Core Principle: Screens Derive from User Stories, Layout Is Separate from Style

Every screen traces back to a User Story — a screen that cannot be justified by a User Story is scope creep. Layout (structure) and style (color/type/icons) are resolved in separate steps deliberately.

---

## Execution Protocol

### Step 0 — Project Folder Setup and Context Load

Confirm a project folder is connected. Load:
1. `docs/features/feature-summary.md` and every `docs/features/F-XX-*.md` — **mandatory**. If absent, halt:
   ```
   ⛔ INSUFFICIENT INPUT
   No approved Feature Breakdown found at docs/features/.
   Run /prd-to-features (and complete HITL Feature Review) before /design-ui.
   ```
2. `ai-context/project-constitution.md` — extract frontend framework.
3. `ai-context/coding-standards.md` — component/naming conventions.
4. `ai-context/design-system.md` — if present, treat as **authoritative**.
5. `ai-context/test-plan.md` — **optional**. E2E scenarios only, as priority signal.

Then scan the project folder for corroborating evidence: Tailwind/CSS config, existing component dirs, brand assets.

---

### Step 1 — Design Preference Elicitation

**1a — Report `design-system.md` and scan findings first, before asking anything.**

**1b — Ask only for what neither source already answered** (max 4 questions per AskUserQuestion call):
1. Brand color(s)
2. Typography
3. Iconography
4. Layout leaning
5. Theming (Light / Dark / Both)
6. Reference anchor ("Should this feel like a specific product?")
7. Accessibility baseline (skip if `design-system.md` answers it)

---

### Step 2 — Screen and Component Inventory (Structural Layer)

Derive screens from User Stories' Acceptance and Behavioral Expectations.

For each screen:
```
SCR-[NNN]: [Screen Name]
Owning Feature(s) : F-XX [, F-XX]
User Stories       : US-XX.N
Purpose            : [One sentence]
States             : Empty | Loading | Error | Populated | [feature-specific]
Primary Actions    : [list]
Data Displayed     : [list]
```

Identify shared/reusable components (CMP-NNN). Build the Navigation / IA Map.

**This step must not reference colors, fonts, or icons.**

---

### Step 3 — Generate Layout Options (Low-Fidelity, Structural Only)

For layout-defining screens, produce **2–3 structurally distinct concepts** in grayscale HTML.
Save to `docs/design/mockups/layout-option-[a|b|c].html`.

---

### Step 4 — HITL Layout Selection Checkpoint

Present concepts via `AskUserQuestion`. If user selects a merge, restate and confirm.

---

### Step 5 — Apply Visual Style and Render Polished Mockups

Using Step 1 preferences and selected layout:
- Render all screens with real color/typography/iconography.
- Cap at **8 screens** for this pass; list skipped screens explicitly.
- All mockups share one stylesheet: `docs/design/mockups/style.css`.
- Wire real cross-screen navigation (`href` links between mockup files).
- Wire each Primary Action to reveal its target state (modal open, tab switch, etc.).
- Honor accessibility baseline: semantic HTML, visible focus states, sufficient contrast.

Save to `docs/design/mockups/[SCR-NNN]-[slug].html`.

---

### Step 6 — Write the UI Design Document and Wire Feature Files

**6a — Compose `docs/design/ui-design.md`:**

```markdown
# UI Design — [Project Name]

**Generated:** [YYYY-MM-DD] | **Skill:** `/design-ui` | **Status:** Draft — Awaiting HITL Review

---

## Design Preferences

| | |
|---|---|
| **Design system source** | [...] |
| **Color palette** | [...] |
| **Typography** | [...] |
| **Iconography** | [...] |
| **Theming** | [...] |
| **Accessibility baseline** | [...] |
| **Reference anchor** | [...] |
| **Selected layout pattern** | [...] |

---

## Navigation & IA Map

[Tree from Step 2]

---

## Screen Inventory

| Screen ID | Name | Feature(s) | States | Mockup |
|---|---|---|---|---|
| SCR-001 | [Name] | F-XX | Empty, Loading, Populated | [Link](./mockups/SCR-001-slug.html) |

---

## Component Inventory

| Component ID | Name | Used On | Purpose |
|---|---|---|---|

---

## Screen Detail

### SCR-001 — [Screen Name]

**Owning Feature(s):** F-XX | **User Stories:** US-XX.N
**Purpose:** [sentence]
**States:** [list]
**Primary Actions:** [list]
**Data Displayed:** [list]
**Mockup:** [Link](./mockups/SCR-001-slug.html)

[Repeat per screen]
```

**6b — Save to both locations:**
1. `docs/design/ui-design.md`
2. `ai-context/ui-design.md` (identical copy)

**6c — ⭐ Write Screen References back into feature files:**

For each feature F-XX, open `docs/features/F-XX-[slug].md` and append (or overwrite) a
`## Screen References` section at the end of the file:

```markdown
## Screen References
*(Generated by /design-ui — do not edit manually)*

| Screen ID | Screen Name | Mockup Path |
|-----------|-------------|-------------|
| SCR-NNN   | [Name]      | docs/design/mockups/SCR-NNN-[slug].html |
| SCR-NNN   | [Name]      | docs/design/mockups/SCR-NNN-[slug].html |
```

Rules:
- Include only screens whose `Owning Feature(s)` includes this F-XX.
- If a screen is structural-only (no rendered mockup), note `(structural-only — no mockup rendered)`.
- If the section already exists, overwrite it entirely — never append to a stale section.
- Mockup path is relative to the project root (e.g. `docs/design/mockups/SCR-001-dashboard.html`).

**6d — Print summary:**

```
✅ UI Design Generated
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Saved to: docs/design/ui-design.md  |  ai-context/ui-design.md
Mockups:  docs/design/mockups/ ([N] screens rendered, [N] structural-only)
Feature files updated: [N] F-XX-*.md files (Screen References section written)

Screens    : [N]  (across [N] features)
Components : [N] shared components identified
Layout     : [selected/merged concept]

Skipped from mockup pass (structural-only): [SCR-IDs, or "None"]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 7 — HITL UI Design Review Checkpoint

```
⛔ HITL — UI DESIGN REVIEW

Confirm all 10 items before proceeding to /feature-to-issues:

Structure
  □ 1. Screen inventory covers every User Story's needed views
  □ 2. Navigation/IA map matches expected user flows
  □ 3. Selected layout pattern is approved

Visual
  □ 4. Color palette, typography, and iconography match brand expectations
  □ 5. Mockups reflect approved layout and style
  □ 6. Component inventory has no obvious missing shared components

Polish
  □ 7. Theming choice is correctly reflected
  □ 8. Mockups consistent with ai-context/design-system.md (or from-scratch confirmed)
  □ 9. Screens are genuinely click-through: every nav link and Primary Action is wired

Feature File Wiring
  □ 10. Every F-XX-*.md that owns screens has a ## Screen References section with correct mockup paths

Type YES to approve and proceed to /feature-to-issues.
Type REVISE [SCR-NNN or item number] [change] to update before approving.
```

**If YES:** State "✅ UI Design Approved." Then /start continues to /feature-to-issues.

**If REVISE:** Apply the change, regenerate the affected mockup(s), re-present the checkpoint.

> This is the mandatory HITL gate between UI design and issue generation. Agents assist. Humans approve.

---

## Design Guardrails

### NEVER
- Invent a brand-critical decision without evidence or explicit user input.
- Re-ask questions `ai-context/design-system.md` already answers.
- Skip the accessibility baseline.
- Skip Step 3 multi-option generation.
- Mix layout and style into the same round of options.
- Leave a Primary Action inert (no visible response to click).
- Leave a Navigation/IA Map relationship as a visual-only element.
- Leave a feature file without a `## Screen References` section when it owns screens.

### ALWAYS
- Load `ai-context/design-system.md` in Step 0 if it exists, and treat as authoritative.
- Trace every screen to at least one Feature ID and User Story ID.
- Cap the styled-mockup pass at 8 screens and list what was skipped.
- Wire every screen-to-screen link and every Primary Action's state-reveal.
- Save both `docs/design/ui-design.md` and `ai-context/ui-design.md`.
- Write `## Screen References` back into every feature file that owns screens (Step 6c).
- End every run with the HITL UI Design Review Checkpoint.

---

## ID Conventions

| Prefix | Type | Format |
|--------|------|--------|
| `SCR-` | Screen (distinct navigable view) | `SCR-001`, `SCR-002`… |
| `CMP-` | Shared/reusable component | `CMP-001`, `CMP-002`… |

Zero-padded to 3 digits, globally unique. Downstream artifacts must reference these IDs verbatim.

---

## Downstream Compatibility

| System | Compatibility |
|---|---|
| `/constitution` (`design-system.md`) | ✅ Consumed as authoritative upstream input |
| `/feature-to-issues` | ✅ Reads SCR-NNN IDs and mockup paths from F-XX-*.md `## Screen References` section (primary) and `ai-context/ui-design.md` (fallback) |
| Ralph-impl | ✅ Mockups describe intended structure/style for UI layer |
