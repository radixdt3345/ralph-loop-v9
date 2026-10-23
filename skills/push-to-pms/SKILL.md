---
name: "ralph-loop-v9:push-to-pms"
slug: push-to-pms
description: >
  Called by /start (Step 8). Do not invoke directly — type /start to begin.
  Reads ai-context/issues.json and creates all epics and issues in the chosen platform
  (GitHub by default, also GitLab, Azure DevOps, Jira, Linear). Writes task_status.md.
---

# /push-to-pms — Issue Publisher

**Inputs:** `ai-context/issues.json` ✓, `ai-context/pms-map.json` (if resuming), platform credentials from /platform-setup

**Outputs:** Issues in platform + `ai-context/pms-map.json` + updated `task_status.md`

---

## Supported Platforms

| Platform | Epic Type | Child Type | Link Type |
|----------|-----------|------------|-----------|
| GitHub Issues | Milestone | Issue (with sub-issues) | "Closes #N" / tasklist |
| GitLab | Epic | Issue | Epic link |
| Azure DevOps | Epic Work Item | User Story + Task | Parent-child |
| Jira | Epic | Story + Sub-task | Epic link |
| Linear | Project / Cycle | Issue | Parent link |

---

## Creation Order

1. Create all Epics first (milestones/epic work items)
2. Create child issues in dependency order (Wave 1 first)
3. Set parent-child relationships
4. Set dependency links ("blocked by" / "depends on")
5. Apply labels: `layer:DB`, `layer:API`, ..., `wave:1`, `domain:[name]`, `agent:ralph-impl`, etc.
6. Assign to project board / milestone

---

## GitHub Implementation

For each Epic → create GitHub Milestone:
```
Title: [EPIC-NN] [Feature Name]
Description: [from ai-context/issues.json epic title]
```

For each Issue → create GitHub Issue:
```
Title: [DOMAIN-LAYER-NNN] [description]
Body: [from issues.json body field — full markdown]
Milestone: [corresponding milestone]
Labels: [layer, wave, domain, agent labels]
```

For sub-issue linking: use GitHub's native sub-issue API to link child issues to their Epic issue (in addition to milestone grouping).

For dependency linking: add `Depends on #[N]` in the issue body (recognized by many GitHub project management extensions).

---

## `ai-context/pms-map.json`

```json
{
  "platform": "github",
  "repo": "[owner/repo]",
  "created": "[ISO timestamp]",
  "milestones": {
    "EPIC-01": { "number": 1, "url": "..." }
  },
  "issues": {
    "AUTH-DB-001": { "number": 42, "url": "...", "state": "open" },
    ...
  }
}
```

---

## Update `task_status.md`

After all issues are created, update the Platform Issue column:

```markdown
| Task ID | Issue ID | Type | Status | GitHub Issue | Blocked By |
| T-001 | AUTH-DB-001 | Development | PENDING | #42 | — |
```

---

## Failure Handling

- If an issue fails to create: retry once, then log to `docs/05_dependencies.md` under "PMS Creation Errors" and continue.
- If rate limited: wait with exponential backoff.
- Save `pms-map.json` even if some items failed — partial maps are valid.
- Never re-create an issue that already exists in pms-map.json (idempotent).

---

## Print on completion

```
✅ Push to [Platform] complete
Epics: [N] | Issues: [N] | Failed: [N]
View board: [URL]
```
