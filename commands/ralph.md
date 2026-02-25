---
description: Ralph Loop — split a plan into subtasks, autonomously implement and review each, report back
argument-hint: <plan, task list, or what you want to build>
---

Ralph Loop: decompose a plan into subtasks, autonomously implement each with iterative review, run independent subtasks in parallel, and report back everything you did, every decision you made, and everything you couldn't figure out.

## Pre-compute Context

```bash
echo "=== GIT STATUS ===" && git status --short 2>/dev/null || echo "Not a git repo"
echo "=== CURRENT BRANCH ===" && git branch --show-current 2>/dev/null || echo "Unknown"
echo "=== TEST RUNNER ===" && (test -f pytest.ini && echo "pytest" || test -f pyproject.toml && grep -q pytest pyproject.toml && echo "pytest" || test -f package.json && node -e "const p=require('./package.json'); p.scripts?.test ? console.log('npm test') : process.exit(1)" 2>/dev/null || test -f Makefile && grep -q "^test:" Makefile && echo "make test" || echo "No test runner detected")
echo "=== PLAYWRIGHT ===" && (test -f playwright.config.ts && echo "playwright.config.ts found" || test -f playwright.config.js && echo "playwright.config.js found" || (test -f package.json && grep -q playwright package.json && echo "Playwright in package.json") || echo "No Playwright detected")
echo "=== DEV SERVER ===" && (test -f package.json && node -e "const p=require('./package.json'); const s=p.scripts||{}; console.log(s.dev||s.start||'none')" 2>/dev/null || echo "No dev server detected")
echo "=== VIBE CONTEXT ===" && (test -f .vibe/understanding.md && echo "Understanding: exists" && head -20 .vibe/understanding.md || echo "Understanding: none — consider running /vibe:init")
echo "=== VIBE RISKS ===" && (test -f .vibe/risks.md && echo "Risks baseline: exists" || echo "Risks baseline: none")
echo "=== VIBE LEARNINGS ===" && (test -f .vibe/learnings.md && cat .vibe/learnings.md || echo "No learnings yet")
echo "=== RALPH STATE ===" && (test -f .vibe/work/ralph.md && echo "Previous ralph state found:" && cat .vibe/work/ralph.md || echo "No previous ralph state")
```

---

# PHASE 1: DECOMPOSE

Break the plan into discrete subtasks and map their dependencies.

## Step 1: Explore

Spawn an **explorer** subagent via the Task tool:

```
Use the Task tool with subagent_type="vibe:explorer" (or "general-purpose" if unavailable) to:
- Read .vibe/understanding.md and .vibe/learnings.md for project context
- Explore the codebase areas relevant to [PLAN]
- Identify the current architecture, patterns, and conventions
- Map which files/modules relate to which parts of the plan
- Flag areas of uncertainty or ambiguity
- If Context7 is available, look up unfamiliar library/framework APIs
```

## Step 2: Present Subtask Breakdown

In the **main conversation**, present the decomposition:

```
RALPH LOOP — TASK BREAKDOWN

Plan: [1-2 sentence summary]

Subtasks (in execution order):

  1. [Subtask name]
     What: [description]
     Files: [list]
     Depends on: [nothing / subtask N]
     Confidence: [HIGH — I know exactly what to do / MEDIUM — mostly clear, may need assumptions / LOW — significant ambiguity]

  2. [Subtask name]
     What: [description]
     Files: [list]
     Depends on: [nothing / subtask N]
     Confidence: [HIGH / MEDIUM / LOW]

  ... (repeat for each subtask)

Parallelism:
  - [Subtasks X and Y] can run in parallel (independent)
  - [Subtask Z] must wait for [Subtask X]

Quality gates:
  - Tests must pass after each subtask
  - No CRITICAL review issues allowed
  - Max 3 review iterations per subtask before moving on

Questions I already have (will block these subtasks if unanswered):
  1. [Question] — blocks subtask N
  2. [Question] — blocks subtask N
  (or: No blocking questions — ready to go fully autonomous)
```

**STOP and wait for user approval.**

The user should:
- Confirm the breakdown is correct
- Answer any blocking questions
- Adjust subtask order or grouping if needed
- Say "go" to start the autonomous loop

---

# PHASE 2: THE RALPH LOOP (autonomous)

**Once the user says go, do NOT stop until all subtasks are done or stuck.**

No asking for approval. No pausing for confirmation. No "does this look right?" Work through every subtask you can. Skip what you can't. Document everything.

## Tracking State

Create/update `.vibe/work/ralph.md` before starting:

```markdown
# Ralph Loop State
Plan: [summary]
Started: [timestamp]
Status: IN PROGRESS

## Subtasks
| # | Name | Status | Iterations | Notes |
|---|------|--------|------------|-------|
| 1 | [name] | PENDING | 0 | |
| 2 | [name] | PENDING | 0 | |
```

Update this file as you go. This is your source of truth.

## Execution Rules

1. **Pick the next subtask** — choose from subtasks whose dependencies are all DONE. If multiple are ready and independent, run them in parallel.

2. **For each subtask, loop:**

   **a. Implement** — Spawn an **engineer** subagent:
   ```
   Use the Task tool with subagent_type="vibe:engineer" (or "general-purpose" if unavailable) to:
   - Implement subtask: [name and description]
   - Files to change: [list]
   - Patterns to follow: [from explorer findings]
   - Read .vibe/understanding.md for project conventions
   - Context from completed subtasks: [what was already done and any relevant details]
   - If Context7 is available, look up library APIs before using them
   - If something is ambiguous: make your best judgment call, document the decision, and keep going
   ```

   **b. Review + Test** — Spawn **reviewer** and **tester** in parallel:

   **Reviewer subagent:**
   ```
   Use the Task tool with subagent_type="vibe:reviewer" (or "general-purpose" if unavailable) to:
   - Review changes for subtask: [name]
   - Check correctness: trace code paths, edge cases, error handling
   - Check security: injection, secrets, auth
   - Check reliability: external service failures, unhandled exceptions
   - Verify library APIs are used correctly (look up docs via Context7 if available)
   - Output: issues with severity (CRITICAL/WARNING/SUGGESTION), verdict (PASS/NEEDS FIXES)
   ```

   **Tester subagent:**
   ```
   Use the Task tool with subagent_type="vibe:tester" (or "general-purpose" if unavailable) to:
   - Run the test suite: [test command]
   - Run linters if available
   - Report pass/fail with counts
   - Output: verdict (PASS/NEEDS FIXES)
   ```

   **c. Evaluate results:**
   - **Both pass** → mark subtask DONE, update ralph.md, move on
   - **Issues found** → increment iteration count, spawn engineer to fix, loop back to (b)
   - **Hit 3 iterations** → mark subtask STUCK, record what's failing and why, move on to next subtask
   - **Ambiguity encountered** → make your best call, record the decision and reasoning in ralph.md, keep going

3. **Parallel execution** — When multiple independent subtasks are ready, spawn their engineer subagents in a single message. After all return, run their reviews in parallel too.

4. **Context passing** — When starting a subtask that depends on a completed one, include a summary of what the prior subtask changed and any decisions it made.

5. **Never stop to ask the user mid-loop.** If you're unsure:
   - If you can make a reasonable assumption → do it, document it
   - If you genuinely cannot proceed → mark the subtask SKIPPED with a clear explanation of what you need, move to the next subtask

---

# PHASE 3: REPORT

When all subtasks are either DONE, STUCK, or SKIPPED, present the full report:

```
RALPH LOOP COMPLETE

Plan: [summary]
Duration: [subtask count] subtasks, [total iterations] total iterations

## Results

  ✓ Subtask 1: [name] — DONE in [N] iterations
    Changes: [files changed]

  ✓ Subtask 2: [name] — DONE in [N] iterations
    Changes: [files changed]

  ✗ Subtask 3: [name] — STUCK after 3 iterations
    Problem: [what's failing and why]
    Last attempt: [what was tried]

  ⊘ Subtask 4: [name] — SKIPPED
    Reason: [what information or decision is needed]

## Decisions I Made
  1. [Decision] — because [reasoning]
  2. [Decision] — because [reasoning]

## Things I Couldn't Figure Out
  1. [Question/blocker] — needed for subtask [N]
  2. [Question/blocker] — needed for subtask [N]

## Review Summary
  Tests: [passing / N failures]
  Critical issues: [0 / N remaining]
  Warnings: [N]
  New risks: [none / list]

## What's Left
  [List of STUCK and SKIPPED subtasks, or "Nothing — all done!"]

Overall: [ALL COMPLETE / N of M subtasks done — need your input on the rest]
```

Update `.vibe/work/ralph.md` with final state.
Update `.vibe/decisions.md` with key decisions.
Update `.vibe/risks.md` with current state.

---

# PHASE 4: CONTINUE (if user responds)

If the user provides follow-up after the report (answering questions, clarifying stuck items, adjusting decisions):

1. Read `.vibe/work/ralph.md` for current state
2. Update subtask definitions based on user input
3. **Go back to Phase 2** — resume the loop for STUCK and SKIPPED subtasks only
4. Do NOT re-do completed subtasks unless the user explicitly asks

This is the Ralph Loop — it keeps going until everything is done or the user says stop.
