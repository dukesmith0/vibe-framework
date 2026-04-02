---
description: Full development workflow - explore, plan, implement, review, verify
argument-hint: <task>
---

Use TodoWrite to track phases.

## Pre-compute
```bash
git status -s 2>/dev/null | head -5; git branch --show-current 2>/dev/null
ls .vibe/ 2>/dev/null; head -5 .vibe/plans.md 2>/dev/null; head -1 .vibe/current.md 2>/dev/null
test -f pytest.ini && echo "test:pytest" || (grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || (test -f vitest.config.ts -o -f vitest.config.js && echo "test:vitest") || (test -f jest.config.ts -o -f jest.config.js && echo "test:jest") || (node -e "const p=require('./package.json');if(p.scripts?.test){console.log('test:'+p.scripts.test);process.exit(0)}else{process.exit(1)}" 2>/dev/null) || (test -f go.mod && echo "test:go") || (test -f Cargo.toml && echo "test:cargo") || echo "test:none"
test -f playwright.config.ts -o -f playwright.config.js && echo "playwright:yes" || (grep -q playwright package.json 2>/dev/null && echo "playwright:yes") || echo "playwright:no"
node -e "try{const p=require('./package.json');console.log('devserver:'+(p.scripts?.dev||p.scripts?.start||p.scripts?.serve||'none'))}catch{}" 2>/dev/null
```
If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phase 1 - Entry routing
Read `.vibe/` context via vibe-context skill. Then route:
- `plans.md` has matching plan? Offer to continue (skip to Phase 4).
- `plans.md` has different plan? Ask: continue existing or start new?
- No plan, no task arg? "Run `/vibe:plan <goal>` or provide a task." Suggest `/vibe:oneshot` for simple tasks.
- Task is "fix #N"? Read bug from `bugs.md`. If Open/Deferred: use as context. If Resolved: inform user. If not found: "Bug #N not found. Open bugs: [list IDs]."
- Task is "fix #RN"? Read risk from `risks.md`. If active: use as context. If Resolved: inform user. If not found: "Risk #RN not found. Active risks: [list IDs]."
- `current.md` shows active task? Ask: resume, abandon, or start new?

## Phase 2 - Explore
Dispatch explorer (`subagent_type="vibe:explorer"` or `"general-purpose"`). Prompt with: task description, understanding.md (architecture, patterns), related risks. Ask: analyze code, identify patterns, flag risks/test gaps, report with file:line refs. If general-purpose fallback: paste understanding.md content directly.

## Phase 3 - Plan + approve
Present 1-3 approaches with trade-offs. Define success criteria as `- [ ]` checkboxes.
**[STOP]** Wait for approval. Record choice as `[USER]` in `decisions.md`. Write plan to `plans.md`. Update `current.md` with phase progress.

## Phase 4 - Implement
Dispatch engineer (`subagent_type="vibe:engineer"` or `"general-purpose"`). Prompt with: task, approach, files to modify, patterns, success criteria. Instruction: "Generate/update debug/ tests. List assumptions and choices in output."
- `DONE`: write assumptions to `decisions.md` as `[CLAUDE]`. Proceed.
- `DONE_WITH_CONCERNS`: read concerns, write assumptions. Address correctness doubts first.
- `BLOCKED`: present to user. 3-fix rule: do not attempt fix #4.
- `NEEDS_CONTEXT`: provide info, re-dispatch.

## Phase 5 - Review + test
Dispatch reviewer + tester **in parallel (single message)**:
- **Reviewer** (`vibe:reviewer`): Stage 1: spec compliance. Stage 2: code quality/bugs/security (CRITICAL/HIGH/MEDIUM/LOW). Stage 3: functionality. Stage 4: risk scan vs baseline. Stage 5: simplifications.
- **Tester** (`vibe:tester`): project tests + debug/ suite + linters. Playwright if frontend changes.

## Phase 6 - Review gate
- CRITICAL/HIGH: **[STOP]** fix required. User approves simplifications individually.
- MEDIUM: present to user, approve or fix.
- LOW: report only.
If fixes needed: re-dispatch engineer, re-review. Max 3 cycles.

## Phase 7 - Update .vibe/ + verify
- `decisions.md`: [USER/CLAUDE] decisions + assumptions
- `bugs.md`: bugs found (next ID, impact level)
- `risks.md`: scan + baseline update. Resolved risks to Resolved section.
- `understanding.md`: if architecture changed
- `debug/`: regression tests for resolved bugs
- `current.md`: progress
Check each success criterion. If any unmet, return to Phase 4.

## Phase 8 - Complete
Clear `current.md` to `# No active task`. Archive plan to `decisions.md` Plan Archive. Clear `plans.md`. Present summary: changes, decisions, tests, risk delta.
