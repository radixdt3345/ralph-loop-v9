---
name: "ralph-loop-v9:status"
slug: status
description: >
  Prints current build progress from task_status.md. Call any time during the
  build loop to see counts by status, the next pending task, and any blocked
  tasks. Not part of the automated pipeline — a user-callable utility.
---

# /status — Forge Build Progress

Run immediately. Read `task_status.md` and print a summary.

---

## Output Format

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FORGE STATUS — [Project Name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total tasks:    [N]
✅ COMPLETE:    [N] ([X]%)
🔄 IN_PROGRESS: [N]
⏳ PENDING:     [N]
❌ BLOCKED:     [N]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Next task:
  [T-NNN] [DOMAIN-LAYER-NNN] — [description]
  GitHub: #[issue number]

In progress:
  [T-NNN] [description] (Ralph-[impl/test/e2e])

Blocked (needs attention):
  [T-NNN] [description] — [reason/label from GitHub]

Build complete when: [N] tasks remaining
Estimated waves: [list remaining waves]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If all tasks are COMPLETE:
```
✅ BUILD COMPLETE — Run /production-readiness to launch.
```

If task_status.md does not exist:
```
No build in progress. Run /start to begin.
```
