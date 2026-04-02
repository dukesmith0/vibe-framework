---
description: End-to-end workflow with minimal stops - clarify, implement, review, present
argument-hint: <task>
---

Use TodoWrite to track progress through all phases.

## Pre-compute

```bash
# Core
git status -s 2>/dev/null | head -5
ls .vibe/ 2>/dev/null
head -1 .vibe/current.md 2>/dev/null

# Testing
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(test -f vitest.config.ts -o -f vitest.config.js && echo "test:vitest") || \
(test -f jest.config.ts -o -f jest.config.js && echo "test:jest") || \
(node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"

# Frontend
test -f playwright.config.ts && echo "playwright:yes" || test -f playwright.config.js && echo "playwright:yes" || \
(grep -q playwright package.json 2>/dev/null && echo "playwright:yes") || echo "playwright:no"
node -e "try{const p=require('./package.json');console.log('devserver:'+(p.scripts?.dev||p.scripts?.start||p.scripts?.serve||'none'))}catch{}" 2>/dev/null
```

If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phase 1 - Pre-compute

Run bash block. Read `.vibe/` context. Check `current.md` for active task (conflict prevention: if active, ask resume/abandon/new).

## Phase 2 - Clarify

Dispatch explorer subagent (`subagent_type="vibe:explorer"` or `"general-purpose"`):

Prompt with:
- The task description
- Understanding.md content (architecture, components, patterns)
- Instruction: "Analyze codebase for this task. Return a structured report: 1) What I think the user wants, 2) What files/components this involves, 3) Assumptions I'm making, 4) Questions before proceeding."

Present explorer's report to user verbatim.

## Phase 3 - Confirm

**[STOP]** User confirms understanding, answers questions, corrects assumptions.
Record user's confirmed understanding as `[USER]` decision in `decisions.md`.

## Phase 4 - Plan + Implement

- Plan internally. Write to `plans.md` with success criteria. Update `current.md`.
- Dispatch engineer subagent (`subagent_type="vibe:engineer"` or `"general-purpose"`):
  - Prompt with: task, approach, files to change, patterns to follow, success criteria
  - Instruction: "Implement following project patterns. Generate/update debug/ tests. List any assumptions and approach choices in your output."
- Handle engineer status (DONE/DONE_WITH_CONCERNS/BLOCKED/NEEDS_CONTEXT). Write assumptions to `decisions.md` as `[CLAUDE]`.

## Phase 5 - Test

Auto-generate/update `debug/` tests for changed code. Run full `debug/` suite + project tests.

## Phase 6 - Review

Dispatch reviewer + tester **in parallel (single message)**:
- Reviewer: two-stage (spec compliance then quality) + bug review + functionality + risk scan vs baseline
- Tester: project tests + debug/ suite + linters + Playwright if applicable

## Phase 7 - Review gate

**[STOP IF CRITICAL/HIGH]** Pause on critical/high issues. User decides fix or continue.
If fix needed: re-dispatch engineer, re-review. Max 3 cycles.

## Phase 8 - Risk scan

Compare against `risks.md` baseline. Report delta per impact level.

## Phase 9 - Simplification

Propose simplifications with before/after snippets for recently changed code only.
**[STOP]** User approves individually.
Rules: preserve exact functionality, don't touch untouched code, don't add features/abstractions.

## Phase 10 - Update .vibe/

- `bugs.md`: bugs found with next sequential ID and impact level
- `risks.md`: scan results with baseline comparison. Move resolved to Resolved section.
- `decisions.md`: user confirmations [USER], Claude choices [CLAUDE], assumptions
- `current.md`: progress
- `understanding.md`: if architecture changed
- `debug/`: regression tests for any resolved bugs

## Phase 11 - Present

Summary: changes made, decisions recorded, test results, risk delta.
Clear `current.md`. Archive plan to `decisions.md` Plan Archive. Clear `plans.md`.
