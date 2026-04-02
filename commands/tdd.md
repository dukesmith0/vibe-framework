---
description: Test-driven development - write tests first, lock contract, implement until green
argument-hint: <feature>
---

Use TodoWrite to track phases.

## Pre-compute
```bash
git status -s 2>/dev/null | head -5; ls .vibe/ 2>/dev/null; head -1 .vibe/current.md 2>/dev/null
test -f pytest.ini && echo "test:pytest" || (grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || (test -f vitest.config.ts -o -f vitest.config.js && echo "test:vitest") || (test -f jest.config.ts -o -f jest.config.js && echo "test:jest") || (node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || (test -f go.mod && echo "test:go") || (test -f Cargo.toml && echo "test:cargo") || echo "test:none"
```
If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phase 1 - Setup
Read `.vibe/` context via vibe-context skill. Update `current.md`: `# Current Task - TDD: [feature] | Via: /vibe:tdd`

## Phase 2 - Explore
Dispatch explorer (`subagent_type="vibe:explorer"` or `"general-purpose"`). Prompt with feature + understanding.md patterns/tests sections. Instruction: "Find test patterns (describe/it? fixtures? table-driven?), test framework API, where tests should live (colocated or debug/). Report: target files, patterns, framework, test location."

## Phase 3 - Write tests (RED)
Dispatch engineer (`subagent_type="vibe:engineer"` or `"general-purpose"`). Prompt with: feature, explorer findings, test location, project test style. Instruction: "Write ONLY tests. NO implementation. Cover: happy path, error paths, edge cases (empty, boundary, null). Match project test patterns. Record requirement assumptions in decisions.md."

## Phase 4 - Verify failure
Dispatch tester (`subagent_type="vibe:tester"` or `"general-purpose"`): run new tests.
- Must FAIL for the right reason (missing implementation, not typo/import error).
- Wrong failure reason? Fix tests, re-run.

## Phase 5 - Approve tests
**[STOP]** Present tests with what each validates. User approves the contract.

## Phase 6 - Commit tests
Commit tests only. Contract locked. **Tests must NOT be modified after this.**

## Phase 7 - Implement (GREEN)
Dispatch engineer (`subagent_type="vibe:engineer"` or `"general-purpose"`). Prompt with: committed test files, understanding.md patterns. Instruction: "Write minimal code to pass ALL tests. Do not modify tests. List assumptions in output."
Write assumptions to `decisions.md` as `[CLAUDE]`.

## Phase 8 - Iterate
Dispatch tester (`subagent_type="vibe:tester"` or `"general-purpose"`): run all tests (project + debug/ + TDD).
- All pass? Proceed.
- Failures? Engineer fixes implementation (NOT tests), tester re-runs. Max 3 iterations.

## Phase 9 - Update .vibe/ + commit
- `decisions.md`: test assumptions, requirement gaps, implementation choices
- `bugs.md`: bugs found (next ID + impact)
- `debug/`: regression tests for resolved bugs
- `current.md`: clear to `# No active task`
Commit implementation. Summary: tests, files changed, iterations, assumptions.
