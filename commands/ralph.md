---
description: Autonomous multi-task loop - decompose, implement and review each subtask, report results
argument-hint: <plan>
---

Use TodoWrite to track progress.

## Pre-compute
```bash
git status -s 2>/dev/null | head -5; git branch --show-current 2>/dev/null
ls .vibe/ 2>/dev/null; head -10 .vibe/current.md 2>/dev/null
test -f pytest.ini && echo "test:pytest" || (grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || (test -f vitest.config.ts -o -f vitest.config.js && echo "test:vitest") || (test -f jest.config.ts -o -f jest.config.js && echo "test:jest") || (node -e "const p=require('./package.json');if(p.scripts?.test){console.log('test:'+p.scripts.test);process.exit(0)}else{process.exit(1)}" 2>/dev/null) || (test -f go.mod && echo "test:go") || (test -f Cargo.toml && echo "test:cargo") || echo "test:none"
test -f playwright.config.ts -o -f playwright.config.js && echo "playwright:yes" || (grep -q playwright package.json 2>/dev/null && echo "playwright:yes") || echo "playwright:no"
```
If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phase 1 - Setup
Read `.vibe/` context. Check `current.md`:
- Active ralph with STUCK/BLOCKED? Offer to resume.
- Active non-ralph task? Ask: resume, abandon, or start new.

## Phase 2 - Decompose
Dispatch explorer (`subagent_type="vibe:explorer"` or `"general-purpose"`). Prompt with plan/goal + understanding.md. Instruction: "Break into subtasks. Each: name, description, files, dependencies, confidence (HIGH/MEDIUM/LOW), complexity (S/M/L). Order by dependency chain."

## Phase 3 - Approve
**[STOP]** Present subtask table: `| # | Subtask | Confidence | Deps | Size |`. Flag LOW confidence. User says "go" to start.
Write plan and ralph state table to `current.md`.

## Phase 4 - The Loop

**AUTONOMOUS. Once "go", do NOT stop. No approvals. No pauses. Make assumptions, document them, keep going.**

Per ready subtask (deps satisfied):

1. Mark IN_PROGRESS in `current.md`
2. **Implement:** dispatch engineer (`vibe:engineer` or `"general-purpose"`). Scene-setting: subsystems, prior subtask changes, patterns, acceptance criteria.
   - Independent subtasks: `run_in_background: true`, dispatch multiple in one message. Wait for completion notification.
   - Overlapping files: `isolation: "worktree"` instead (one at a time). Never combine both on same dispatch.
   - Sequential deps: one at a time, pass prior context forward.
3. **Self-review:** engineer reviews own work before reporting.
4. **Generate tests:** update `debug/` for changed code.
5. **Review:** dispatch reviewer + tester **in parallel (single message)**. Reviewer: spec compliance then quality. Tester: full project + debug/ suite.
6. **Fix loop:** if issues, same engineer fixes, re-run BOTH reviewer AND tester. Max 3 iterations.
7. **Evaluate:** DONE / DONE_WITH_CONCERNS / STUCK (3 iter) / BLOCKED (missing info)
8. **Record immediately:** decisions `[CLAUDE]` + assumptions to `decisions.md`. Bugs to `bugs.md` (next ID + impact). Risks to `risks.md` (next ID).
9. Update `current.md` subtask table.

## Phase 5 - Report
```
| # | Subtask | Result | Iter | Notes |
DONE_WITH_CONCERNS items: [specific concerns]
Could not complete: [STUCK/BLOCKED details]
Decisions: [count] | Bugs: [IDs] | Risk delta: +N/-N
```

## Phase 6 - Finalize
All DONE? Archive plan to `decisions.md` Plan Archive. Update `understanding.md` if arch changed. Clear `current.md` to `# No active task`.
STUCK/BLOCKED remain? Keep ralph state in `current.md` for resume.

## Phase 7 - Resume
If user responds: resume for STUCK/BLOCKED only. Don't redo completed. User may provide new context.
