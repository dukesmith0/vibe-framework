---
description: Show available vibe commands and suggest next action based on project state
---

Check project state:
```bash
test -d .vibe && echo "vibe:initialized" || echo "vibe:none"
test -f .vibe/current.md && head -5 .vibe/current.md
git status -s 2>/dev/null | head -1
```

Show commands grouped by use case:

**Getting started:** `/vibe:init` (analyze codebase), `/vibe:help` (this)
**Planning:** `/vibe:plan <goal>` (brainstorm + plan), `/vibe:ask <question>` (query context)
**Building:** `/vibe:go <task>` (guided workflow), `/vibe:tdd <feature>` (test-first), `/vibe:oneshot <task>` (minimal stops), `/vibe:ralph <plan>` (autonomous multi-task)
**Maintaining:** `/vibe:review` (adversarial review), `/vibe:commit` (smart commit), `/vibe:refresh` (sync after manual changes), `/vibe:note <content>` (quick entry), `/vibe:add <path>` (import docs)

Suggest next action based on state:
- No `.vibe/`? -> "Run `/vibe:init` to analyze your codebase."
- `current.md` shows active task or plan? -> "Active task in progress. Run `/vibe:go` to resume."
- Git changes staged? -> "Changes ready. Run `/vibe:commit` to commit."
- No git? -> "Note: No git repo detected. Most commands work, but commit/push disabled. Run `/vibe:init` to set up."
