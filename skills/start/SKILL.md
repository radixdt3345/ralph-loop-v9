---
name: "ralph-loop-v9:start"
description: >
  Entry point for the Forge pipeline (v6). Run when the user types /start or wants
  to begin a new project. Orchestrates 9 steps in order: discovery (/grill), PRD
  (/write-prd), constitution (/constitution), feature decomposition (/prd-to-features),
  issue generation (/feature-to-issues), platform setup (/platform-setup), push to PMS
  (/push-to-pms), then build (/ralph). Constitution is ratified before features so
  /ralph has coding standards on every task. Also triggers when resuming an in-progress build.
---

# /start — Forge Pipeline Orchestrator (Entry Point)

**This is the only command you need to type.** All other skills (grill, write-prd,
constitution, prd-to-features, feature-to-issues, platform-setup, push-to-pms, ralph)
are called automatically by /start in order. Do not invoke them directly.

Exceptions (standalone only):
- `/constitution amend` — to change a ratified rule
- `/constitution check [path]` — to audit code against the constitution
- `/ralph` — to resume a build that was interrupted after Step 8

Run immediately when triggered. No preamble. No "how can I help". Just act.

**Ownership rule**: /start owns the pipeline ORDER and coordinates which skill runs when.
Each skill owns its own output files. /start does not generate files itself in the lean flow.
Every other stage is DELEGATED — read and follow that skill's SKILL.md. Never
re-implement another skill's logic here.

---

## The Pipeline

```
Step 1   Detect state
Step 2   Discovery                  → /grill
Step 3   PRD                        → /write-prd         [HIL — PRD Review]
Step 4   Constitution               → /constitution      [HIL — Ratification]
Step 5   Feature decomposition      → /prd-to-features   [HIL — Feature Review]
Step 6   UI Design                  → /design-ui         [HIL — UI Design Review]
Step 7   Test plan                  → /write-test-plan   [HIL — Test Plan Review]
Step 8   Issue generation           → /feature-to-issues [HIL — Issue Review]
Step 9   Platform setup             → /platform-setup
Step 10  Push to PMS                → /push-to-pms
Step 11  Build                      → /ralph
```

---

## Step 1 — Detect State

**If `task_status.md` exists and has tasks:**
→ Print: `▶️  Resuming build... reading task_status.md`
→ Run `/ralph` (per its SKILL.md)

**If `docs/` has content but no `task_status.md`:**
→ Print: `📂 Found docs. Rebuilding task_status.md from issues.json...`
→ Regenerate task_status.md from `ai-context/issues.json` (read issue IDs and statuses), then resume at Step 11 (/ralph).

**If `docs/00_discovery.md` exists but nothing else:**
→ Print: `📋 Found confirmed discovery. Continuing from PRD...`
→ Skip to Step 3 (/write-prd).

**If folder is empty:**
→ Print: `🔨 Forge ready. Let's build something.`
→ Continue to Step 2.

---

## Step 2 — Discovery

Run the complete `/grill` skill per its SKILL.md. Do not shortcut it.

`/grill` ends by writing:
- `docs/00_discovery.md` — confirmed project picture (read by all downstream skills)
- Clarity Summary in context — consumed immediately by Step 4 (/write-prd)

After `/grill` completes, return here and continue to Step 3.

---

## Step 3 — PRD

Run `/write-prd` per its SKILL.md. Pass the Clarity Summary from /grill as context.

`/write-prd` writes `docs/prd.md` and `ai-context/project.json`.

⛔ **HIL — PRD Review** (enforced by /write-prd's own gate): User confirms all FR-,
AC-, and PT- IDs. On YES, continue to Step 4.

---

## Step 4 — Constitution

Run `/constitution` per its SKILL.md (CREATE mode).

The only file it needs is `docs/prd.md`. It asks the user 5 quick questions inline
(tech stack, formatter/linter, test framework, auth mechanism, hard constraints)
and derives everything else from the PRD.

`/constitution` writes:
- `CONSTITUTION.md` — the project's supreme engineering law (locked after ratification)
- `ai-context/project-constitution.md` — bridge file read by /ralph on every task
- `ai-context/coding-standards.md` — coding rules read by /ralph on every task

⛔ **HIL — Ratification** (enforced by /constitution's own gate): User ratifies all
5 articles. On YES, continue to Step 5.

---

## Step 5 — Feature Decomposition

Run `/prd-to-features` per its SKILL.md.

Reads only `docs/prd.md`. Writes:
- `docs/features/feature-summary.md`
- `docs/features/F-XX-[slug].md` (one per feature)

⛔ **HIL — Feature Review** (enforced by /prd-to-features): User confirms features,
user stories, waves, and dependencies. On YES, continue to Step 6.

---

## Step 6 — UI Design

Run `/design-ui` per its SKILL.md.

Reads `docs/features/` and `ai-context/project-constitution.md`. Asks the user inline
about colors, typography, iconography, layout, and theming. Generates 2–3 low-fidelity
layout options, user picks one, then produces polished HTML mockups for up to 8 screens.

Writes:
- `docs/design/ui-design.md` + `ai-context/ui-design.md` (screen inventory, component
  inventory, navigation map — SCR-NNN and CMP-NNN IDs)
- `docs/design/mockups/*.html` (click-through HTML mockups with real nav links wired)

⛔ **HIL — UI Design Review** (enforced by /design-ui Step 7): User confirms all 9
checklist items (screens, nav, layout, style, mockups, components, theming, accessibility,
click-through wiring). On YES, /start continues to Step 7.

---

## Step 7 — Test Plan

Run `/write-test-plan` per its SKILL.md.

Reads `docs/prd.md`, `docs/features/`, and `ai-context/ui-design.md`. Asks one
inline question if test framework isn't in coding-standards.md.

Writes:
- `docs/test-plan.md` — full test plan with UT-, IT-, SMK-, RT- suites
- `ai-context/test-plan.md` — compact reference consumed by /feature-to-issues and /ralph

⛔ **HIL — Test Plan Review** (enforced by /write-test-plan): User confirms all
test types are covered, smoke suite is < 2 min, no MUST AC gaps. On YES, continue
to Step 8.

---

## Step 8 — Issue Generation

Run `/feature-to-issues` per its SKILL.md.

Reads `docs/features/` and `ai-context/ui-design.md` and `ai-context/test-plan.md`.
Writes `ai-context/issues.json`. UI-layer issues reference SCR-NNN / CMP-NNN IDs
and mockup paths. Every issue body includes its owned UT-/IT-/SMK- test IDs.

⛔ **HIL — Issue Review** (enforced by /feature-to-issues): User confirms issue
manifest and dependency graph. On YES, continue to Step 9.

---

## Step 9 — Platform Setup

Run `/platform-setup` per its SKILL.md.

Creates the GitHub repo (or GitLab/Azure DevOps) and project board.
Does NOT create issues — that is Step 10.

`/platform-setup` writes `docs/platform_map.md`.

---

## Step 10 — Push Issues to PMS

Run `/push-to-pms` per its SKILL.md.

Reads `ai-context/issues.json`. Creates all epics and issues in the chosen platform.
Writes `ai-context/pms-map.json` and `task_status.md`.

---

## Step 11 — Build

Run `/ralph` per its SKILL.md.

/ralph reads `ai-context/issues.json`, `ai-context/project-constitution.md`,
`ai-context/coding-standards.md`, `ai-context/ui-design.md`,
`ai-context/test-plan.md`, `task_status.md` and dispatches the
parallel multi-agent build. Ralph-impl writes all owned test IDs per task.

---

## Pipeline Status Conventions

Print a status banner at the start of each major step:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FORGE  Step [N]/11  [Step Name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

On HIL gates, print:

```
⛔ HIL [N] — [Gate Name]
[What needs confirming]
Type YES to continue or provide corrections.
```

Never proceed past a HIL gate without an explicit YES or correction from the user.
