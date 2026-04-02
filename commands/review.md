---
description: Five-part adversarial review - code quality, bugs, functionality, risks, simplification
argument-hint: [scope]
---

Use TodoWrite to track progress.

## Pre-compute
```bash
git diff --name-only 2>/dev/null | head -20; git diff --stat 2>/dev/null; git diff --staged --name-only 2>/dev/null | head -10
ls .vibe/ 2>/dev/null; head -3 .vibe/risks.md 2>/dev/null; head -3 .vibe/plans.md 2>/dev/null
test -f pytest.ini && echo "test:pytest" || (grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || (test -f vitest.config.ts -o -f vitest.config.js && echo "test:vitest") || (test -f jest.config.ts -o -f jest.config.js && echo "test:jest") || (node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || (test -f go.mod && echo "test:go") || (test -f Cargo.toml && echo "test:cargo") || echo "test:none"
test -f playwright.config.ts -o -f playwright.config.js && echo "playwright:yes" || (grep -q playwright package.json 2>/dev/null && echo "playwright:yes") || echo "playwright:no"
node -e "try{const p=require('./package.json');console.log('devserver:'+(p.scripts?.dev||p.scripts?.start||p.scripts?.serve||'none'))}catch{}" 2>/dev/null
```
If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phase 1 - Five-part review

Dispatch reviewer + tester **in parallel (single message)**:

**Reviewer** (`subagent_type="vibe:reviewer"` or `"general-purpose"`). Prompt with: changed files (git diff), plans.md, understanding.md patterns, risks.md baseline, bugs.md. Instruction:
"Adversarial five-part review:
1. **Spec compliance:** match intent/plan from plans.md?
2. **Code quality:** patterns, style, error handling, security (validation, auth, secrets, injection)
3. **Bug review:** logic errors, off-by-one, null, race conditions, edge cases, error propagation. Cross-check bugs.md for reintroductions.
4. **Functionality:** does it work as intended? All paths reachable and tested? Integrations correct?
5. **Risk scan:** compare vs risks.md baseline. Report delta per impact level.
Output: `[CRITICAL/HIGH/MEDIUM/LOW] file:line: issue - fix`. Verdict: APPROVE or NEEDS FIXES."

**Tester** (`subagent_type="vibe:tester"` or `"general-purpose"`). Instruction: "Run project tests, debug/ tests, linters. If Playwright detected + frontend changes: `npx playwright test --reporter=list` (check dev server at localhost:3000/5173, start with 60s timeout if down). Skip Playwright for backend-only."

## Phase 2 - Write findings + test gaps

- `bugs.md`: bugs found with next ID and impact
- `risks.md`: new risks with next ID. Resolved risks (file:line gone) to Resolved section with date. Update baseline.
- Check `debug/` for gaps: changed code without tests? Generate missing tests, verify they pass.

## Phase 3 - Review gate

**[STOP IF CRITICAL/HIGH]** Present findings with verdict.

## Phase 4 - Simplification

Propose changes with before/after snippets for recently changed code only. **[STOP]** User approves individually.
Rules: preserve functionality, don't touch untouched code, don't add features/abstractions/renames.
Valid: dead code, redundant conditionals, verbose patterns, duplicated logic.

## Phase 5 - Summary + next steps

Impact counts, verdict, risk delta, test results, simplifications applied.

Based on verdict:
- `APPROVE`: "Ready to commit. Run `/vibe:commit`."
- `NEEDS FIXES` with bugs: "Run `/vibe:go fix #N` or `/vibe:plan` for broader fix."
- `NEEDS FIXES` with risks: "Run `/vibe:go fix #RN` to address."
- Mixed: list each ID. "Fix individually or `/vibe:plan fix review findings` to tackle together."
