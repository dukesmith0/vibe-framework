---
name: vibe-session-end
description: Warns about active tasks and uncommitted .vibe/ changes before session ends
hooks:
  - event: Stop
---

Before session ends, check:

1. Read `.vibe/current.md`. If it contains an active task (not "# No active task"):
   - Warn: "Active task in progress: [task name]. State saved in current.md for resume next session."

2. Check `git status .vibe/` for uncommitted changes:
   - If changes found: "Uncommitted .vibe/ changes detected. Run `/vibe:commit` to persist context."
