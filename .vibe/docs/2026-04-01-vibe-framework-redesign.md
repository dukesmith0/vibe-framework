# Vibe Framework Redesign Spec

## Overview

Ground-up rewrite of the vibe Claude Code plugin. Persistent `.vibe/` project context becomes the backbone. Auto-generated `debug/` test harness validates everything. All commands, agents, and skills rewritten for conciseness, accuracy, and token efficiency. Inspired by superpowers plugin patterns (iron laws, rationalization guards, scene-setting, two-stage review).

Originally created by [Jack Lutz](https://github.com/jwlutz). Forked and extended by [dukesmith0](https://github.com/dukesmith0).

---

## 1. The `.vibe/` Structure

Created by `/vibe:init`, maintained automatically by all workflows, synced manually via `/vibe:refresh`.

```
.vibe/
├── understanding.md   # Architecture, stack, components, patterns, docs index
├── current.md         # Active task state, subagent progress, ralph loop tracking
├── plans.md           # Current plan - approach, tasks checklist, success criteria
├── bugs.md            # Known bugs with status and context
├── risks.md           # Security, maintainability, tech debt risks with baseline tracking
├── future.md          # Backlog - deferred plans, ideas, someday/maybe
├── decisions.md       # Decisions + learnings with date and rationale
└── docs/              # Reference materials (imported via /vibe:add)
```

### File Format

Concise prose with inline notes. Short sentences, human-readable, token-efficient. Append-only formats where possible to minimize merge conflicts.

**understanding.md example:**
```markdown
# my-project
Last: 2026-04-01 | 847 files | TypeScript, Python

## Stack
Express + Passport + JWT, Node 20. React 18 + Vite. Postgres/Prisma. Redis (sessions, queue).

## Architecture
Monorepo: apps/api (REST, /api/v1/), apps/web (feature-folders: features/auth/, features/dashboard/).

## Components
Auth: src/auth/index.ts. Middleware chain + role guards. Deps: users table, Redis.
Dashboard: apps/web/features/dashboard/. WebSocket /ws/metrics for real-time.

## Patterns
- asyncHandler wrapper on all routes
- Colocated tests: foo.ts -> foo.test.ts
- Config: src/config.ts (single load, import everywhere)

## Tests
vitest (apps/web), jest (apps/api). describe/it blocks. Run: `npm test`

## Docs Index
- [API Spec](docs/api-v1-spec.md) - OpenAPI 3.0
- [PRD](docs/product-requirements.md) - Product requirements
```

**current.md example:**
```markdown
# Current Task - Add rate limiting
Started: 2026-04-01 14:30 | Via: /vibe:go | Phase: 8/12 Review

- [x] 1. Pre-compute
- [x] 2. Entry routing
- [x] 3. Explorer analyzed
- [x] 4. Plan presented
- [x] 5. User approved plan
- [x] 6. Plan written
- [x] 7. Engineer: DONE - rate-limit.ts, app.ts
- [ ] 8. Review cycle (in progress)
- [ ] 9-12. Remaining phases

## Ralph State (when active)
| # | Subtask | Status | Iter | Notes |
|---|---------|--------|------|-------|
| 1 | Rate limit middleware | DONE | 1 | Clean |
| 2 | Auth route integration | DONE | 2 | Missed /refresh |
| 3 | Data route integration | IN_PROGRESS | 1 | |
| 4 | Integration tests | PENDING | 0 | Deps: #2,#3 |
```

**current.md rules:** Update at every phase transition. Use the command's phase numbers. One file, one format. When no task active: `# No active task`

**bugs.md example:**
```markdown
# Bugs
Next ID: 5

## Open
#3 [HIGH] Auth token refresh race condition - two concurrent refreshes, second fails. src/auth/refresh.ts:45
#4 [LOW] WebSocket reconnect drops first message - stale count ~2s. src/dashboard/ws.ts:88

## Deferred
#2 [MEDIUM] Legacy CSV import accepts malformed headers. Known limitation, revisit if complaints increase.

## Resolved
#1 [MEDIUM] CSV export unicode crash. Fixed: encoding header in src/export/csv.ts:12. Regression: debug/test_csv.py
```

**Numbering:** Sequential IDs (#1, #2, ...). `Next ID` counter at top. IDs never reused. Max 2 lines per entry.

**Impact levels:** `CRITICAL` (system down, data loss, security breach), `HIGH` (major feature broken, workaround exists), `MEDIUM` (feature degraded, minor user impact), `LOW` (cosmetic, edge case, no workaround needed).

**Statuses:** `Open` (needs fixing), `Deferred` (acknowledged, won't fix now), `Resolved` (fixed, include regression test ref).

**risks.md example:**
```markdown
# Risks
Next ID: R7 | Last scan: 2026-04-01 | Baseline: 1 critical, 2 high, 1 medium, 2 low

## Critical
#R1 [CRITICAL] Hardcoded JWT secret in test config. apps/api/test/setup.ts:3 (found 2026-03-25)

## High
#R2 [HIGH] No rate limiting on /api/v1/auth/login. Brute force risk. (found 2026-03-25)
#R3 [HIGH] processInvoice 142 lines, high change risk. billing/calculate.ts:47 (found 2026-04-01)

## Medium
#R4 [MEDIUM] 14 TODOs across src/. Cluster: src/billing/ (6). (found 2026-03-25)

## Low
#R5 [LOW] 3 unused imports. src/utils/helpers.ts (found 2026-03-25)
#R6 [LOW] [ACCEPTED] 2 console.log in src/debug/. Accepted per decision 2026-04-01.

## Resolved
#R0 [LOW] Dead import in auth/index.ts. Removed 2026-04-01.
```

**Risk numbering:** Sequential IDs (#R1, #R2, ...). `Next ID` counter at top. Per-entry `(found DATE)` replaces separate changelog section. Max 2 lines per entry.

**Impact levels:** `CRITICAL` (active security vulnerability, data exposure), `HIGH` (potential security issue, major maintainability problem), `MEDIUM` (tech debt, moderate maintenance burden), `LOW` (minor code smell, negligible impact).

**Risk statuses:** Active (in impact section), `[ACCEPTED]` (acknowledged, user chose not to fix), Resolved (moved to Resolved section with date).

**plans.md example:**
```markdown
# Current Plan - Add rate limiting to API

Created: 2026-04-01 | Status: In Progress

## Goal
Prevent brute force and abuse on public API endpoints.

## Approach
Express-rate-limit middleware with Redis store. Per-IP limiting on auth routes (10/min), per-user on data routes (100/min).

## Success Criteria
- [ ] Rate limit middleware created and tested
- [ ] Auth routes return 429 after 10 requests/min from same IP
- [ ] Data routes return 429 after 100 requests/min from same user
- [ ] Existing tests still pass
- [ ] No new critical risks introduced

## Tasks
- [x] Research express-rate-limit + rate-limit-redis
- [x] Write failing tests for rate limit behavior
- [ ] Implement middleware in src/middleware/rate-limit.ts
- [ ] Add to auth routes and data routes
- [ ] Run full test suite
- [ ] Review and commit
```

**future.md example:**
```markdown
# Future Plans

- WebSocket auth - currently WS connections don't validate JWT expiry after initial handshake. Needs heartbeat token refresh. Low priority until user count grows.
- Migrate from REST to tRPC - would eliminate API spec maintenance. Blocked until frontend team is ready.
- Add OpenTelemetry tracing - will help debug cross-service latency. Nice-to-have for Q3.
```

**decisions.md example:**
```markdown
# Decisions

## Technical Decisions
2026-04-01 [USER] Redis for rate limit store. Multi-instance needs shared state, already in stack.
2026-03-30 [USER] Use bun, not npm. npm had phantom deps in CI. Applies project-wide.
2026-03-28 [USER] WebSocket on same server. Scale doesn't justify split. Revisit at 10k.
2026-04-01 [CLAUDE] Middleware pattern over route guards. Matches existing auth middleware chain.

## Assumptions
2026-04-01 [CLAUDE] API returns JSON for all endpoints. Basis: all routes use res.json(). Status: ACTIVE
2026-03-30 [CLAUDE] Users table has unique email. Basis: Prisma schema @unique. Status: VERIFIED

## Learned Lessons
2026-04-01 [CLAUDE] Correction: rate-limit-redis needs ioredis, not redis package. Tests failed with redis.
2026-03-30 [USER] Always use bun, not npm. npm caused phantom deps in CI.

## Plan Archive
Completed: Add rate limiting (2026-04-01) | Done
Added express-rate-limit + Redis store. Decisions: Redis over in-memory, middleware over route guards.
```

### Token Efficiency Rules for .vibe/ Files

All commands writing to `.vibe/` files must follow these rules:

1. **One entry, max two lines.** No prose paragraphs. State the fact, then the reason.
2. **Abbreviate paths** when unambiguous: `src/auth/index.ts` -> `auth/index.ts` if no conflict.
3. **No redundant headers.** Don't repeat the file name in the content.
4. **Use IDs for cross-reference.** "See #3" or "See #R2" instead of restating the bug/risk.
5. **Dates as YYYY-MM-DD only.** No timestamps unless tracking within a single day.
6. **Strip filler words.** Not "Rationale: Because the system needs..." but "Why: System needs..."
7. **One sentence rationale.** If you need two, the second is a separate entry.
8. **Per-entry dates** instead of changelog sections. `(found 2026-04-01)` inline.
9. **Status inline** not in separate fields. Append `VERIFIED` or `ACCEPTED` to entry line.
10. **Archive old entries.** When resolved/completed entries exceed 20, move oldest to a `## Archive` section at bottom.

**Target:** Each .vibe/ file should be <100 tokens for a small project, <500 tokens for a large one.

### Auto-Maintenance Rules

`.vibe/` stays current without manual effort:

| Trigger | Files Updated |
|---------|--------------|
| `/vibe:init` | Creates all 7 files + `docs/` folder + `debug/` folder |
| `/vibe:go`, `/vibe:oneshot`, `/vibe:ralph` | `current.md`, `plans.md`, `decisions.md`, `bugs.md`, `risks.md`. Update `understanding.md` if architecture changed. |
| `/vibe:review` | `bugs.md`, `risks.md` (with baseline comparison) |
| `/vibe:commit` | Check diff against `understanding.md`, update if changed. Mark resolved bugs in `bugs.md`. Mark resolved risks in `risks.md`. |
| `/vibe:tdd` | `current.md`, `decisions.md`, `bugs.md` (if bugs found during testing) |
| `/vibe:refresh` | All files - diffs current codebase against stored state |
| `/vibe:note` | Target file (auto-detected or user-specified) |
| `/vibe:plan` | `plans.md` or `future.md` (user chooses). Archive existing plan to `decisions.md` if replacing. |
| `/vibe:add` | `understanding.md` (docs index updated) |

### Staleness & Reconciliation

Auto-maintenance handles common drift scenarios:

**Deleted files:** `/vibe:refresh` and `/vibe:init` (incremental mode) check all components listed in `understanding.md` against the filesystem. Remove entries for files that no longer exist. Remove corresponding `debug/` tests that reference deleted source. Report deletions.

**New entry points:** `/vibe:refresh` scans for new files matching entry point patterns (main.*, index.*, server.*, app.*, routes/*, handlers/*). Add to `understanding.md`, generate corresponding `debug/` tests.

**Dependency changes:** `/vibe:refresh` checks package manifests (package.json, pyproject.toml, go.mod, Cargo.toml) against `understanding.md` Stack section. Update if dependencies added/removed.

**Test framework changes:** If `/vibe:init` or `/vibe:refresh` detects a different test framework than what `debug/` uses, regenerate `debug/` with the new framework. Report the switch.

**Config changes:** `/vibe:refresh` checks env schemas (.env.example, docker-compose.yml, config files) and updates `understanding.md` if configuration requirements changed.

**Architecture change definition:** New top-level directories, new entry points, new framework integrations, new service boundaries = architecture change → update `understanding.md`. File renames, dependency version bumps, config tweaks = NOT architecture changes.

### Review Verdict Gates

Review findings use the same impact levels as bugs/risks:

| Impact | In /vibe:go | In /vibe:commit | In /vibe:review |
|--------|-------------|-----------------|-----------------|
| CRITICAL | STOP. Fix required before proceeding. | Block commit. Must fix first. | STOP. Ask user: fix now or defer? |
| HIGH | STOP. Fix required before proceeding. | Block commit. Must fix first. | STOP. Ask user: fix now or defer? |
| MEDIUM | Present to user. User approves to continue or requests fix. | Allow commit. Log in commit body. | Report. User decides. |
| LOW | Report in summary. No gate. | No gate. | Report. No gate. |

### Risk Baseline Management

Risk baseline = the counts and specific entries in `risks.md` at last scan.

- **Baseline updates after:** `/vibe:commit` (post-commit scan), `/vibe:review` completion, `/vibe:refresh`
- **Baseline does NOT update after:** `/vibe:go` mid-workflow scans (these report delta against stable baseline)
- **Delta format:** "+N high (list specific), -N resolved (list specific), net change per impact level"
- **Resolved detection:** A risk is resolved when the file:line reference no longer matches the pattern. Moved code re-scans at new location.

### Command Conflict Prevention

Only one active task at a time:

- Before starting work, commands check `current.md`. If it shows an active task (e.g., ralph loop IN_PROGRESS), present: "Active task found: [description]. Resume, abandon, or start new?"
- **Resume** → continue where left off
- **Abandon** → archive to `decisions.md` with outcome "abandoned", clear `current.md`
- **Start new** → archive current, clear, proceed with new task
- `/vibe:ask`, `/vibe:note`, `/vibe:add`, `/vibe:help`, `/vibe:refresh` are read/append operations - they never conflict and can run anytime.

### current.md Lifecycle

- **Created** when a workflow command starts (go, oneshot, ralph, tdd)
- **Updated** at each phase transition with progress, subagent status, decisions
- **Cleared** (reset to empty header) when task completes, is abandoned, or is replaced
- **Archived** task summary goes to `decisions.md` with date, outcome, key decisions
- Between tasks, `current.md` contains only: `# No active task`

### Decisions, Assumptions & Learnings

`decisions.md` captures four sections. One line per entry, max two lines for complex ones. Tagged by source.

**Entry format:** `DATE [USER|CLAUDE] Brief decision. Why in one sentence.`

**Source tags:** `[USER]` = user explicitly decided. `[CLAUDE]` = Claude/agent chose or assumed.

**Sections:**
- **Technical Decisions**: Choices about architecture, tools, patterns. Both user and Claude.
- **Assumptions**: Things Claude assumed without explicit user direction. Status: ACTIVE / VERIFIED / CORRECTED.
- **Learned Lessons**: Corrections, gotchas, operational tips. Created when Claude corrects itself (auto-updates original assumption to CORRECTED) or when user teaches something.
- **Plan Archive**: Completed/abandoned plans. One entry per plan: title, status, summary, key decisions.

**When to auto-record (per command, per phase):**

| Command | Phase | What to record | Section |
|---------|-------|---------------|---------|
| `/vibe:plan` | 6 (user chooses) | User's chosen approach | Technical Decisions [USER] |
| `/vibe:go` | 5 (approval) | User's plan choice | Technical Decisions [USER] |
| `/vibe:go` | 7 (implement) | Claude's approach choices, API assumptions | Technical Decisions [CLAUDE], Assumptions |
| `/vibe:go` | 8 (review) | Reviewer pattern recommendations | Learned Lessons |
| `/vibe:go` | 12 (complete) | Plan archive entry | Plan Archive |
| `/vibe:oneshot` | 3 (confirm) | User's confirmed understanding | Technical Decisions [USER] |
| `/vibe:oneshot` | 4 (implement) | Claude's choices, assumptions | Technical Decisions [CLAUDE], Assumptions |
| `/vibe:oneshot` | 10 (update) | Bugs found -> bugs.md with ID and impact | bugs.md |
| `/vibe:ralph` | per subtask | Claude's per-subtask choices, assumptions | Technical Decisions [CLAUDE], Assumptions |
| `/vibe:ralph` | 7 (report) | Plan archive entry | Plan Archive |
| `/vibe:tdd` | 3 (write tests) | Test design assumptions, requirement gaps | Assumptions, Learned Lessons |
| `/vibe:tdd` | 9 (update) | What was learned during TDD cycle | Learned Lessons |
| `/vibe:review` | 2-4 (review) | Pattern-as-convention recommendations | Learned Lessons |
| Any phase | Self-correction | Correction + update original assumption to CORRECTED | Learned Lessons |

**Superseding:** `DATE [USER] Supersedes: "old decision". New choice. Why.`

### Simplification Categories

What counts as a valid simplification (for /vibe:review and /vibe:oneshot):

**Yes - simplify:** Dead code, redundant conditionals, verbose patterns replaceable with clearer idioms, duplicated logic extractable to shared function, unnecessary type assertions, overly complex error handling
**No - don't touch:** Variable/function naming (subjective), code organization/file structure, performance optimizations, adding abstractions "for clarity", anything in untouched code

### Plan Lifecycle

1. **Created** - `/vibe:plan` or `/vibe:go` writes to `plans.md` with success criteria
2. **Active** - workflows track progress in `current.md`, check off tasks in `plans.md`
3. **Completed** - summary archived to `decisions.md`, `plans.md` cleared, `current.md` reset
4. **Abandoned/Deferred** - archived to `decisions.md` with outcome, optionally moved to `future.md`
5. **Replaced** - if new plan starts while one exists, archive current with outcome first

**Archival format** (written to decisions.md):
```markdown
## Completed: [Plan Title] (YYYY-MM-DD)
Status: Done | Abandoned | Deferred
Summary: [1-2 sentences of what was accomplished or why stopped]
Key decisions: [bullet list of decisions made during execution]
```

### Docs Folder & `/vibe:add`

Reference materials are imported via `/vibe:add <path>`, not manually placed:
- Accepts single files or entire folders (preserves directory structure)
- Copies originals to `.vibe/docs/`
- **Heavy format handling:** For PDFs, images, and non-text formats, a companion `.md` is auto-generated with extracted content, same filename. Commands read the `.md` version. Original preserved.
- **Large file handling:** `/vibe:add` asks the user: "This file is [X pages/KB]. Import full text or summary only?" Default: under ~10 pages → full text. Over ~10 pages → summary with key sections. User can override. Recommends splitting very large docs into focused sections.
- Updates docs index in `understanding.md` with one-line description per file
- Indexed docs become available for all future planning, decisions, and implementation

---

## 2. The `debug/` Test Harness

Auto-generated comprehensive test suite at project root. Created by `/vibe:init`, automatically maintained by all code-generating workflows.

```
debug/
├── conftest.py / setup.ts    # Shared fixtures, test utilities (framework-dependent)
├── test_auth.py              # Tests for auth module
├── test_api_routes.py        # Tests for API routes
├── test_dashboard.py         # Tests for dashboard features
└── ...                       # Mirrors source structure
```

### Framework Selection

Detected per language during `/vibe:init`:

| Language | Framework | Detection |
|----------|-----------|-----------|
| Python | pytest | `pyproject.toml`, `pytest.ini`, `setup.cfg`, existing `conftest.py` |
| TypeScript/JavaScript | vitest or jest | `vitest.config.*`, `jest.config.*`, `package.json` test scripts |
| Go | go test | `go.mod` presence |
| Rust | cargo test | `Cargo.toml` presence |
| Other | Best available for language | Scan for existing test infrastructure |

If project already has a test framework configured, `debug/` uses the same one. Never introduce a competing framework.

**If no test framework detected:** `/vibe:init` asks the user: "No test framework detected. Which would you like to use?" and recommends the best option based on the project's languages:
- Python project → recommend pytest (fast, fixtures, plugins)
- TypeScript/JavaScript → recommend vitest (fast, ESM native, Vite compatible) or jest (mature, widely supported)
- Go → `go test` (built-in, no choice needed)
- Rust → `cargo test` (built-in, no choice needed)
- Multi-language → recommend one per language, ask for each

User can override any recommendation. The chosen framework is recorded in `understanding.md` Test Conventions section.

### What Gets Tested

Tests are headless, comprehensive, and auto-generated:

**Functionality tests:**
- Core business logic paths - happy path + error paths
- API endpoints - request/response validation, status codes, error handling
- Data transformations - input/output verification
- Integration points - service boundaries, database queries, external API calls

**Edge case tests:**
- Boundary values - empty inputs, max lengths, zero/null/undefined
- Concurrency - race conditions identified in `bugs.md`
- Error propagation - exceptions bubble correctly, error messages are useful

**Security tests:**
- Input validation - SQL injection, XSS, command injection attempts
- Auth/authz - unauthorized access, privilege escalation, token expiry
- Secrets - no hardcoded credentials in source (mirrors risk-detection patterns)

**Regression tests:**
- Every bug in `bugs.md` gets a regression test when fixed
- Tests verify the fix AND that the original failure mode is caught

### Auto-Maintenance Rules

| Trigger | debug/ Action |
|---------|--------------|
| `/vibe:init` | Create initial test suite based on codebase analysis |
| `/vibe:go` (after implementation) | Add/update tests for changed code. Run full suite. |
| `/vibe:tdd` | Tests written to `debug/` (TDD cycle uses debug/ as home) |
| `/vibe:oneshot` (after implementation) | Add/update tests, run suite |
| `/vibe:ralph` (per subtask) | Add tests per subtask, run after each |
| `/vibe:review` | Run full `debug/` suite as part of review. Add tests for any gaps found. |
| `/vibe:commit` | Run `debug/` suite before committing. Block if failures. |
| `/vibe:refresh` | Check for untested new code, generate missing tests |
| Bug resolved | Add regression test to `debug/` |

### Integration with Existing Tests

- `debug/` tests coexist with project's existing test suite
- Both suites run during review and commit workflows
- `debug/` never duplicates tests that already exist in the project
- If project uses colocated tests (e.g. `foo.test.ts` next to `foo.ts`), `debug/` focuses on integration/edge/security tests that complement them
- Test runner commands: run project tests first, then `debug/` tests. Both must pass.

### Test Quality Standards

- Each test has a clear name describing the behavior being verified
- Tests are independent - no shared mutable state, no execution order dependency
- Tests are fast - mock external services, use in-memory databases where appropriate
- Tests include assertions with specific expected values, not just "doesn't throw"
- Failed tests report: what was expected, what happened, relevant file:line in source

### Playwright Integration

When Playwright is detected (`playwright.config.ts/js` or `playwright` in `package.json`):
- `debug/` includes browser tests for frontend changes
- Run with `npx playwright test --reporter=list`
- Skip if changes are backend-only or config-only
- Start dev server if needed (detect from `package.json` `scripts.dev` or `scripts.start`)
- Test: page navigation, console errors, network requests, visual regressions

---

## 3. Commands

### Final Command Set (13 commands)

| Command | Purpose | Lines Target |
|---------|---------|-------------|
| `/vibe:init` | Create `.vibe/` + `debug/`, analyze codebase | 70-90 |
| `/vibe:plan <goal>` | Brainstorm → questions → plan → execute now or defer | 60-80 |
| `/vibe:go <task>` | Guided: explore → plan → implement → review+risk scan → verify | 90-110 |
| `/vibe:tdd <feature>` | Write tests → lock contract → implement until green | 70-90 |
| `/vibe:oneshot <task>` | Minimal-stop end-to-end | 80-100 |
| `/vibe:ralph <plan>` | Autonomous multi-task loop with 3-iteration limit | 95-110 |
| `/vibe:review [scope]` | Adversarial review + risk scan + debug/ tests + simplification | 70-85 |
| `/vibe:commit [hint]` | Smart commit/push, .vibe/ sync, debug/ gate, optional PR | 55-65 |
| `/vibe:ask <question>` | Answer using .vibe/ context + codebase search | 20-25 |
| `/vibe:note [type] <content>` | Quick entry to any .vibe/ file, auto-detect type | 25-30 |
| `/vibe:add <path>` | Import file/folder to docs/, convert heavy formats, index | 30-40 |
| `/vibe:refresh` | Sync .vibe/ + debug/ after manual code changes | 45-55 |
| `/vibe:help` | Show commands, suggest next action based on project state | 20-30 |

**Removed from original:** `learn` (→ decisions.md), `log` (→ ask), `risks` (→ review), `status` (→ ask)

### Command Design Patterns

**Pre-compute modules (composable, not copy-pasted):**

Commands compose only the modules they need:

```
_core:     git status, branch, .vibe/ existence, current.md + plans.md headers
_testing:  test framework detection (precedence: pytest.ini → pyproject.toml → package.json → Makefile → setup.cfg → go.mod → Cargo.toml)
_review:   git diff (staged + unstaged), changed file list, risks.md baseline counts
_frontend: playwright config detection, dev server detection (scripts.dev → scripts.start → scripts.serve)
_commit:   staged diff, unstaged diff, untracked files, recent 5 commits, branch, remotes
```

| Command | Modules Used |
|---------|-------------|
| `/vibe:init` | `_core`, `_testing` |
| `/vibe:go` | `_core`, `_testing`, `_frontend` |
| `/vibe:tdd` | `_core`, `_testing` |
| `/vibe:oneshot` | `_core`, `_testing`, `_frontend` |
| `/vibe:ralph` | `_core`, `_testing`, `_frontend` |
| `/vibe:review` | `_core`, `_review`, `_testing`, `_frontend` |
| `/vibe:commit` | `_commit`, `_testing` |
| `/vibe:plan` | `_core` |
| `/vibe:ask` | `_core` |
| `/vibe:refresh` | `_core`, `_testing`, `_frontend` |
| `/vibe:note`, `/vibe:add`, `/vibe:help` | `_core` (lightweight) |

**Phase structure - numbered checklist, not prose:**
```
1. Pre-compute - [bash block]
2. Explore - explorer subagent: [specific task]
3. Plan - present 1-3 options with success criteria checkboxes
4. [STOP] Wait for user approval
5. Implement - engineer subagent: [specific task]
6. Review + Test + Risk Scan - reviewer + tester in parallel, run debug/ suite
7. Update .vibe/ - decisions, bugs, risks, current
8. Verify - check success criteria, present summary
```

**Subagent dispatch pattern:**
- Always include fallback: `subagent_type="vibe:explorer" (or "general-purpose" if unavailable)`
- Always spawn reviewer + tester in a **single message** for parallel execution
- Scene-setting context for each dispatch: what subsystems are affected, dependencies, how to know if done
- When dispatching dependent subtasks, include summary of what prior subtask changed

**Stop point patterns:**
- `[STOP]` - unconditional, always pause for user
- `[STOP IF CRITICAL/HIGH]` - pause if CRITICAL or HIGH impact issues found, ask user whether to fix or continue
- Autonomous phases (ralph loop) - no stops once user says "go"

**Error fallbacks:**
- No `.vibe/`? → "No vibe context found. Run /vibe:init to analyze this codebase."
- Push fails? → Check if remote is ahead, pull if needed, retry
- Pre-commit hook fails? → Display error. Ask user to fix or run `/vibe:review` to auto-fix. Re-stage, create NEW commit (never amend).
- No git? → Commands still work. Use timestamps instead of commits for tracking. Disable push/PR features. Note in help output.
- No changes to commit? → "No changes to commit." Exit cleanly.
- `/vibe:add` path not found? → Report error, suggest similar paths via fuzzy match if possible.
- Risk scan fails? → "Risk scan incomplete - manual review recommended." Continue workflow.
- Playwright timeout? → Check if dev server running (curl localhost:3000/localhost:5173). Start with 60s timeout. If still fails, skip Playwright tests with warning.

**Subagent fallback context:**
When `vibe:[agent]` is unavailable and falling back to `"general-purpose"`: the general-purpose agent won't have the `vibe-context` skill loaded. Manually include relevant `.vibe/` content in the dispatch prompt - paste understanding.md patterns, current plan, and any task-specific context directly.

### Command Details

#### `/vibe:init`
1. Pre-compute - detect languages, test frameworks, existing `.vibe/`, existing `debug/`
2. **Version control check:**
   - Check if project is in a git repo (`git rev-parse --git-dir`)
   - Check if remote exists (`git remote -v`)
   - If no git: ask user "Initialize git repository? [yes/no]" → if yes, run `git init`
   - If git but no remote: ask "Create a GitHub repository? [yes/no]" → if yes, ask public/private, then `gh repo create` with appropriate flags and set as origin
   - If git + remote: confirm and continue
3. If `.vibe/` exists, do incremental update (acts like refresh). If not, create full structure.
4. If `debug/` exists with user-written tests, ask: "Merge existing tests into generated suite or start fresh?" Default: merge.
5. Scan codebase structure (exclude: `node_modules, .git, __pycache__, .venv, venv, build, dist, .pytest_cache, debug/`)
6. Top-level analysis first - identify architecture type, languages, entry points, high-level organization
7. Deep dive on key directories - components, patterns, data flow, dependencies
8. Identify risks - scan using risk-detection patterns with impact levels, write to `risks.md`
9. Index any existing files in `.vibe/docs/` - generate `.md` companions for heavy formats
10. Create `debug/` test harness - if existing framework detected, use it. If no framework found, ask user which to use with recommendation based on project languages. Generate initial comprehensive tests.
11. Run `debug/` suite to verify baseline
12. Create all `.vibe/` files. Initialize `current.md` as `# No active task`, `plans.md`, `bugs.md`, `future.md`, `decisions.md` as empty with headers.
13. For empty/new projects: create `.vibe/` structure with minimal content. `understanding.md`: "New project, no code analyzed yet." All other files: empty with headers. `debug/`: empty with setup file only.
14. Output: summary - git status (repo/remote), component count, risk counts by impact level, patterns identified, test count, framework used

#### `/vibe:plan <goal>`
1. Pre-compute - .vibe/ context
2. Read `understanding.md` + `future.md` (goal may relate to existing future plan - if match found, surface it)
3. Ask clarifying questions one at a time (prefer multiple choice)
4. Propose 2-3 approaches with trade-offs and recommendation
5. Present plan with tasks as checkable items and success criteria as `- [ ]` checkboxes
6. Record user's chosen approach as `[USER]` decision in `decisions.md`
7. [STOP] Ask user: "Execute now or save for later?"
   - **Execute now** → archive existing plan if any (→ `decisions.md` Plan Archive with outcome), write to `plans.md`, update `current.md`, then **automatically begin `/vibe:go` workflow** (skip go's explore+plan phases since plan already exists - jump to go phase 6: implement)
   - **Save for later** → append to `future.md` with one-line summary. Return to user prompt.

#### `/vibe:go <task>`
1. Pre-compute - git state, .vibe/ context, test runner, playwright, dev server
2. **Entry routing:**
   - If `plans.md` has active plan matching task → offer to continue it (skip to phase 6)
   - If `plans.md` has active plan NOT matching task → ask: continue existing plan or start new?
   - If no plan and no task argument → "No plan or task specified. Run `/vibe:plan <goal>` to create a plan, or provide a task: `/vibe:go <task>`." If the user then provides a simple/small task (single file change, quick fix, small addition), suggest: "This looks like a quick task. Run `/vibe:oneshot <task>` instead for a faster workflow, or continue with `/vibe:go` for full planning."
   - If task provided but no plan → proceed to phase 3 (explore+plan)
   - If task references a bug ID (e.g., "fix #3") → read bug details from `bugs.md`, use as task context, proceed to phase 3
   - If task references a risk ID (e.g., "fix #R2") → read risk details from `risks.md`, use as task context, proceed to phase 3
3. Explore - explorer subagent: analyze relevant code, patterns, risks, test gaps
4. Plan - present 1-3 approaches with trade-offs. Define success criteria as `- [ ]` checkboxes.
5. [STOP] Wait for user approval of plan. Record chosen approach as `[USER]` decision.
6. Write approved plan to `plans.md`. Update `current.md` with task state.
7. Implement - engineer subagent with scene-setting context. Follow patterns from `understanding.md`. Record assumptions as `[CLAUDE]` entries.
8. **Comprehensive review cycle** (all run automatically after implementation):
   a. Auto-generate/update `debug/` tests for changed code
   b. Run full `debug/` suite + project tests
   c. **Code review** - reviewer subagent (spec compliance → code quality) in parallel with tester
   d. **Bug review** - reviewer checks for logic errors, edge cases, race conditions, error handling gaps
   e. **Functionality review** - does implementation actually satisfy the success criteria from step 4?
   f. **Risk scan** - compare current risks against `risks.md` baseline, report delta (added/resolved/net per impact level)
   g. **Code simplification** - propose simplifications for recently changed code (before/after snippets)
   h. Output: `[CRITICAL/HIGH/MEDIUM/LOW] file:line: issue - fix` + `Verdict: APPROVE / NEEDS FIXES`
9. [STOP IF CRITICAL/HIGH] Present findings. User approves simplifications individually.
10. Update `.vibe/`:
    - `decisions.md`: decisions [USER/CLAUDE] + assumptions from phases 5 and 7
    - `bugs.md`: bugs found with next sequential ID and impact level
    - `risks.md`: new scan with baseline update. Move resolved risks (file:line no longer matches) to Resolved section.
    - `current.md`: progress at phase 10
    - `understanding.md`: if architecture changed
    - `debug/`: regression tests for any bugs resolved during implementation
11. Verify - check each success criterion from step 4. If any unmet, go back and fix before completing.
12. Complete - clear `current.md`, archive plan summary to `decisions.md` Plan Archive, clear `plans.md`.

**3-fix rule:** If engineer fails to pass review after 3 fix attempts, STOP. This is likely an architectural problem. Present findings to user for discussion instead of attempting fix #4.

**Decision auto-detection:** If Claude corrects itself or discovers something important during any phase, proactively record to `decisions.md`.

#### `/vibe:tdd <feature>`
1. Pre-compute - git state, .vibe/ context, test framework detection (precedence: `pytest.ini` -> `pyproject.toml` -> `package.json` -> `Makefile` -> `setup.cfg`)
2. Update `current.md` with task state (phase 1/10, TDD mode)
3. Explore - explorer subagent: analyze target code, find test patterns, identify test framework APIs
4. Write tests - engineer subagent writes failing tests to `debug/` (or project test location if colocated pattern). Record any requirement assumptions in `decisions.md`.
5. Run tests - tester confirms they FAIL for the right reason (not typo/import error)
6. [STOP] Present tests to user for approval
7. Commit tests - lock the test contract
8. Implement - engineer subagent writes minimal code to pass tests. Record implementation assumptions as `[CLAUDE]`.
9. Iterate - tester runs tests, loop until green (max 3 iterations before escalating to user)
10. Update `.vibe/`:
    - `current.md`: progress at phase 10
    - `decisions.md`: test design assumptions, requirement gaps discovered, implementation choices
    - `bugs.md`: any bugs found during testing with ID and impact
    - `debug/`: regression tests for any resolved bugs
11. Commit implementation

#### `/vibe:oneshot <task>`
1. Pre-compute - git state, .vibe/ context, test runner, playwright, dev server
2. Clarify - explorer subagent analyzes. Present back to user: "What I think you want", "What this involves", "Assumptions I'm making", "Questions before I proceed"
3. [STOP] Confirm understanding, answer questions, correct assumptions
4. Plan internally, write to `plans.md`. Implement - engineer subagent.
5. Auto-generate/update `debug/` tests. Run full suite.
6. Review + Test - reviewer + tester in parallel (single message). Two-stage: spec then quality.
7. [STOP IF CRITICAL/HIGH] Pause on critical issues
8. Risk scan - baseline comparison, report delta per impact level
9. Simplification - propose simplifications with before/after snippets. [STOP] Approve individually. Only apply approved. Rules: only recently changed code, preserve functionality, no refactoring untouched code.
10. Update `.vibe/`:
    - `bugs.md`: write any bugs found with next sequential ID and impact level
    - `risks.md`: update with scan results and baseline comparison
    - `decisions.md`: record user confirmations as [USER], Claude choices as [CLAUDE], assumptions
    - `current.md`: update progress
    - `understanding.md`: if architecture changed
    - `debug/`: add regression tests for any resolved bugs
11. Present - summary of changes, decisions recorded, test results, risk delta

#### `/vibe:ralph <plan>`
1. Pre-compute - git state, .vibe/ context. Check `current.md` for existing ralph state (resume capability).
2. Decompose - explorer subagent breaks plan into subtasks with dependencies, confidence levels (HIGH/MEDIUM/LOW), and estimated complexity
3. [STOP] Present breakdown with subtask table for approval. Flag LOW confidence subtasks for user attention.
4. Write plan to `plans.md`. User says "go" to start autonomous loop.
5. **The Loop** (autonomous - no stops until done or stuck):
   - Pick next ready subtask (dependencies satisfied)
   - Update `current.md` with subtask state table
   - **Implement** - engineer subagent with scene-setting: what subsystems affected, what prior subtasks changed, acceptance criteria for this subtask
   - **Self-review** - engineer performs own review before reporting: requirements met? patterns matched? tests comprehensive? anything to flag?
   - Auto-generate/update `debug/` tests for changed code
   - **Two-stage review** - reviewer subagent:
     - Stage 1: Spec compliance (does it match the subtask spec?)
     - Stage 2: Code quality + bugs + security
   - **Test** - tester subagent in parallel with reviewer (single message). Run project tests + `debug/` suite.
   - If review finds issues: same engineer fixes, reviewer re-checks (bounded: max 3 iterations total per subtask)
   - Evaluate: **DONE** / **DONE_WITH_CONCERNS** (completed but has doubts) / **STUCK** (3 iteration limit) / **BLOCKED** (missing info)
   - Record Claude decisions as `[CLAUDE]` and assumptions in `decisions.md` immediately (not batched)
   - Write bugs to `bugs.md` with next ID and impact immediately when found
   - Write risks to `risks.md` immediately when found (don't wait for loop end)
   - For dependent subtasks: include summary of what prior subtask changed and decisions made
   - Dispatch independent subtasks in parallel where possible (use background agents or worktree isolation for overlapping files)
   - Track iteration count per subtask in `current.md`
   - Once user says go, do NOT stop until all subtasks are done or stuck. No asking for approval. No pausing for confirmation. Make reasonable assumptions, document them, keep going.
6. Report - comprehensive results:
   - Subtask status table with check/x/circle symbols for DONE/STUCK/BLOCKED
   - DONE_WITH_CONCERNS items listed with specific concerns
   - Decisions made (archived to `decisions.md`)
   - Assumptions made (recorded in `decisions.md` Assumptions section)
   - Bugs found (written to `bugs.md` with impact level)
   - Risk delta (updated in `risks.md`)
   - Things that couldn't be figured out (for STUCK/BLOCKED items)
7. Update `.vibe/` - everything touched during loop
8. **Resume** - if user responds, resume loop for STUCK/BLOCKED items only. Do NOT redo completed subtasks unless explicitly asked.

#### `/vibe:review [scope]`
Comprehensive five-part review: code quality, bugs, functionality, risks, and simplification.

1. Pre-compute - git diff, changed files, .vibe/ risks baseline, plans.md (for spec compliance), playwright config, dev server
2. **Code review** - reviewer subagent (adversarial):
   - **Stage 1: Spec compliance** - do changes match intent/plan from `plans.md`?
   - **Stage 2: Code quality** - patterns, style, naming, duplication, error handling
   - **Stage 3: Security** - input validation, auth, secrets, injection vectors
   - Output: `[CRITICAL/HIGH/MEDIUM/LOW] file:line: issue - fix`
3. **Bug review** - reviewer continues:
   - Logic errors, off-by-one, null/undefined handling, race conditions
   - Edge cases: empty inputs, boundary values, concurrent access
   - Error propagation: do exceptions bubble correctly? Are error messages useful?
   - Cross-reference with `bugs.md`: are any known bugs reintroduced?
4. **Functionality review** - reviewer continues:
   - Does implementation actually do what was intended?
   - Are all code paths reachable and tested?
   - Do integrations with other components work correctly?
5. Test - tester subagent in parallel (single message): run project tests + `debug/` suite + linters
6. Playwright - if detected AND changes include frontend: run `npx playwright test --reporter=list`. Start dev server if needed (check localhost:3000/5173 first, start with 60s timeout if down). Skip if backend-only changes.
7. **Risk scan** - compare against `risks.md` baseline. Report delta per impact level: added (list), resolved (list), net change.
8. Write findings to `bugs.md` (bugs with impact level) and `risks.md` (risks with baseline update)
9. Check `debug/` for gaps - any changed code without corresponding tests? Generate missing tests.
10. [STOP IF CRITICAL/HIGH] Present all findings with verdict: `APPROVE / NEEDS FIXES`
11. **Code simplification** - propose changes with before/after snippets for recently changed code only. [STOP] User approves individually. Rules: preserve exact functionality, don't refactor untouched code, don't add features/abstractions/renamed variables.
12. Summary - impact counts, verdict, risk delta, test results, simplifications applied
13. **Next step guidance** based on verdict:
    - `APPROVE`: "Ready to commit. Run `/vibe:commit`."
    - `NEEDS FIXES` with bugs: "Bugs recorded in bugs.md. Run `/vibe:go fix #N` to address specific bugs, or `/vibe:plan` to plan a broader fix."
    - `NEEDS FIXES` with risks: "Risks updated in risks.md. Run `/vibe:go fix #RN` to address specific risks. Address CRITICAL/HIGH before committing."
    - Mixed bugs + risks: List each with its ID. "Run `/vibe:go fix #3` or `/vibe:go fix #R2` to address individually, or `/vibe:plan fix review findings` to tackle them together."

#### `/vibe:commit [hint]`
1. Pre-compute - `git status -s`, `git diff --staged`, `git diff`, `git log --oneline -5`, `git branch --show-current`, `git remote -v | head -2`
2. Run `debug/` suite - if failures, report and do NOT proceed. Tests must pass before commit.
3. Run project tests - same gate.
4. Analyze changes - what changed and why, group into logical commit
5. Check - does diff affect architecture/patterns? → update `understanding.md`
6. Check - does diff resolve any bugs in `bugs.md`? → mark resolved, add regression test to `debug/` if not already present
7. Check - does diff resolve any risks in `risks.md`? (file:line pattern no longer matches) → move to Resolved section with date
8. Draft commit message - concise, follows project style from recent commits. First line under 72 chars. Body if "why" isn't obvious. Use conventional commits only if no clear existing style. **Do NOT add Co-Authored-By lines. User's attribution setting handles this.**
9. [STOP] Present staged changes + draft message for approval
10. Commit and push. If push fails: check if remote is ahead, pull if needed, retry. If pre-commit hook fails: fix issue, re-stage, create NEW commit (never amend).
11. Offer PR creation if on feature branch (not main/master)

#### `/vibe:ask <question>`
1. Read `.vibe/` context (understanding.md, relevant files based on question)
2. Check docs index - load relevant docs if question relates to indexed material
3. Search actual codebase - Grep/Read for specific details, file:line references, code snippets
4. Answer with citations: file:line references, code snippets where helpful
5. If no `.vibe/`: "No vibe context found. Run /vibe:init to analyze this codebase."

#### `/vibe:note [type] <content>`
1. If `type` provided, use it. If omitted, auto-detect from content:
   - Mentions error, crash, broken, fails → `bug`
   - Mentions security, vulnerability, hardcoded, exposed → `risk`
   - Mentions decided, chose, use X not Y, always/never → `decision`
   - Mentions later, someday, should eventually, nice-to-have → `future`
   - If ambiguous, ask user
2. Append entry to target file with date
3. Confirm: "Added to [file]: [summary]"

Valid types: `bug` → `bugs.md`, `risk` → `risks.md`, `decision` → `decisions.md`, `future` → `future.md`

#### `/vibe:add <path>`
1. Validate path exists (file or folder)
2. Copy to `.vibe/docs/`, preserving folder structure for directories
3. For each file:
   - Text files (`.md`, `.txt`, `.json`, `.yaml`, etc.) → copy as-is
   - Heavy formats (PDF, images, `.docx`, `.pptx`, etc.) → copy original + generate companion `.md`
   - Large files (>10 pages or >50KB text): ask user "Import full text or summary only?" Default: under 10 pages → full, over 10 → summary. Recommend splitting very large docs.
4. Update docs index in `understanding.md` - one-line description per file
5. Report: files added, conversions made, index updated

#### `/vibe:refresh`
For use after manual code changes outside vibe commands:
1. **Reconcile understanding.md:**
   - Check all components listed - remove entries for deleted files
   - Scan for new entry points (main.*, index.*, server.*, app.*, routes/*, handlers/*)
   - Check package manifests for dependency changes - update Stack section
   - Check config files (.env.example, docker-compose, etc.) - update if changed
   - If >30% of listed files changed, do full re-scan (mini init). Otherwise delta-update.
2. **Reconcile debug/:**
   - Validate all test imports - remove tests referencing deleted source files
   - Check if test framework changed - if so, regenerate debug/ with new framework
   - Scan for untested new code - generate missing tests
3. Scan for new/changed files in `.vibe/docs/` - update index, generate `.md` companions if needed
4. Re-scan risks - update `risks.md` with baseline comparison
5. Run `debug/` suite to verify current state
6. Report: what changed, files removed, tests added/removed, new risks, test results

#### `/vibe:help`
1. Check if `.vibe/` exists
2. Show commands grouped by use case:
   - **Getting started:** `init`, `help`
   - **Planning:** `plan`, `ask`
   - **Building:** `go`, `tdd`, `oneshot`, `ralph`
   - **Maintaining:** `review`, `commit`, `refresh`, `note`, `add`
3. Suggest next action based on state:
   - No `.vibe/`? → "Run `/vibe:init` to analyze your codebase"
   - Has plan in `plans.md`? → "You have an active plan. Run `/vibe:go` to continue"
   - Changes staged? → "Changes ready. Run `/vibe:commit` to commit"
   - Stale understanding? → "Codebase may have changed. Run `/vibe:refresh`"

---

## 4. Agents

### Shared Context Loading (`vibe-context` skill)

One skill handles all `.vibe/` context loading, tailored per agent role. Eliminates repeated preambles.

| Agent | Loads from .vibe/ |
|-------|-------------------|
| explorer | understanding.md (full), risks.md, bugs.md, plans.md, current.md, docs index |
| engineer | understanding.md (patterns + components + test conventions), plans.md, current.md, decisions.md (recent 5) |
| reviewer | understanding.md (patterns + test conventions), risks.md (baseline), bugs.md |
| tester | understanding.md (test conventions only), bugs.md (for regression checks) |

**API/Library lookup chain (all agents):**
1. If Context7 MCP available → `resolve-library-id` then `query-docs` for unfamiliar APIs
2. If Context7 unavailable → WebSearch for documentation
3. Read source code comments as last resort
4. **Never guess an API. Always verify before using.**

### Agent Design (15-25 lines each)

Each agent file contains ONLY role-specific instructions. Format:

```markdown
---
name: [role]
description: [one line - Use when ...]
subagent_type: vibe:[role]
tools: [tool list]
---

## Role
[2-3 sentences]

## Process
[4-6 step checklist]

## Output
[Exact format + status codes]

## Guards
[2-3 rationalization blockers]
```

### Agent Specifications

#### Explorer
- **Tools:** Read, Glob, Grep, Bash, WebSearch, WebFetch, LSP
- **Role:** Investigates codebases to map architecture, patterns, and concerns. Goes broad first (structure), then deep (specific files).
- **Process:** Structure scan → component analysis → data flow tracing → pattern identification → concern flagging → verify frameworks via docs
- **Output:** Findings report: key files, connections, patterns, frameworks (verified), concerns. File:line references throughout.
- **Guards:**
  - Don't skim - read files completely before reporting
  - Don't assume framework behavior - verify via Context7/docs
  - Don't omit concerns to seem positive - flag everything

#### Engineer
- **Tools:** Read, Glob, Grep, Bash, WebSearch, WebFetch, LSP
- **Role:** Implements code changes following project patterns exactly. Research-first: reads context, verifies APIs, then codes.
- **Process:** Read .vibe/ context → verify unfamiliar APIs (Context7 → WebSearch) → implement matching existing style → generate/update `debug/` tests → run tests → list changes with reasoning
- **Output:** Status: `DONE` / `DONE_WITH_CONCERNS` / `BLOCKED` / `NEEDS_CONTEXT` + files changed with reasoning. If DONE_WITH_CONCERNS, list specific doubts.
- **Guards:**
  - Don't guess APIs - look them up before using
  - Don't skip existing patterns - match them exactly
  - Don't add unrequested features or "improvements"
  - **3-fix rule:** If 3 fix attempts fail, report BLOCKED - it's architectural, not a bug

#### Reviewer
- **Tools:** Read, Glob, Grep, Bash, WebSearch, WebFetch, LSP
- **Role:** Adversarial code review. Two-stage: spec compliance first, then code quality. Verify library API correctness via Context7/docs.
- **Process:** Read patterns from context → Stage 1: check spec compliance → Stage 2: check correctness, security, style → verify APIs used correctly → verdict
- **Output:** `[CRITICAL/HIGH/MEDIUM/LOW] file:line: issue - suggestion` + `Verdict: APPROVE / NEEDS FIXES`
- **Guards:**
  - Don't rubber-stamp - if uncertain, it's a WARNING not a pass
  - Don't suggest style changes that contradict project patterns
  - Don't skip edge cases because the happy path works

#### Tester
- **Tools:** Read, Glob, Grep, Bash, WebSearch, WebFetch (no LSP)
- **Role:** Runs project tests + `debug/` suite + linters. Reports results with failure details.
- **Process:** Detect test framework (precedence: `pytest.ini` → `pyproject.toml` → `package.json` → `Makefile` → `setup.cfg`) → run project tests → run `debug/` tests → run linters → report
- **Output:**
  ```
  Project Tests: PASS/FAIL (ran/passed/failed)
  Debug Tests: PASS/FAIL (ran/passed/failed)
  Lint: PASS/FAIL/NOT CONFIGURED
  Verdict: READY / NEEDS FIXES
  [If failures: exact output, file:line in source]
  ```
- **Guards:**
  - Don't report PASS if any test was skipped or errored
  - Don't skip linters if they exist
  - Don't mask flaky tests - report them as WARNING

---

## 5. Skills

### `vibe-context` (Core Skill)

**Trigger:** Auto-loads when any command or agent needs `.vibe/` context. Triggers on code questions, implementation tasks, reviews, planning.

**Behavior:**
1. Check if `.vibe/` exists. If not: "No vibe context found. Run /vibe:init."
2. Load files based on caller role (see agent context table)
3. Read docs index from `understanding.md`. If current task relates to an indexed doc, load that doc on-demand.
4. API lookup chain: Context7 (`resolve-library-id` → `query-docs`) → WebSearch → source comments. Never guess.
5. Provide loaded context to caller.

**Target size:** 35-45 lines.

### `risk-detection` (Rewritten)

**Trigger:** During review, risk scanning, code changes, or when evaluating security.

**Language-agnostic patterns (all projects):**
- Hardcoded secrets: `(api_key|password|secret|token|private_key)\s*=\s*["'][^"']+["']`
- `.env` files committed to git
- Functions >100 lines
- Nesting >4 levels deep
- TODO/FIXME/HACK/XXX clusters (>3 in one file)
- Files >500 lines

**JavaScript/TypeScript:**
- `eval()`, `new Function()`
- `innerHTML`, `dangerouslySetInnerHTML`
- Unvalidated `req.params`, `req.query`, `req.body`
- `console.log` in production code

**Python:**
- `exec()`, `eval()`
- `cursor.execute` with string formatting (SQL injection)
- Bare `except:` (swallows all errors)
- `print()` in production code

**Output:** Always file:line references with impact level (CRITICAL/HIGH/MEDIUM/LOW). Writes to `risks.md` grouped by impact with baseline comparison.

**Target size:** 30-40 lines.

---

## 6. Plugin Metadata & Branding

### plugin.json
```json
{
  "name": "vibe",
  "description": "Persistent project context, auto-generated test harness, and disciplined development workflows for Claude Code.",
  "version": "3.0.0",
  "author": {
    "name": "dukesmith0",
    "url": "https://github.com/dukesmith0"
  },
  "contributors": [
    {
      "name": "Jack Lutz",
      "url": "https://github.com/jwlutz",
      "note": "Original creator of claude_code_framework"
    }
  ],
  "repository": "https://github.com/dukesmith0/vibe-framework",
  "homepage": "https://github.com/dukesmith0/vibe-framework",
  "license": "MIT",
  "keywords": [
    "workflow", "context", "planning", "subagents", "testing", "risk-scanning",
    "tdd", "decision-tracking", "autonomous", "code-review", "bugs", "debug"
  ]
}
```

### marketplace.json
```json
{
  "name": "vibe-marketplace",
  "description": "Persistent project context, auto-generated test harness, and disciplined development workflows for Claude Code.",
  "owner": {
    "name": "dukesmith0",
    "url": "https://github.com/dukesmith0"
  },
  "metadata": {
    "description": "Persistent .vibe/ project context that survives across sessions. Auto-generated debug/ test harness. Subagent-orchestrated workflows for planning, implementation, review, and testing. Auto-maintained understanding, bug tracking, risk scanning, and decision logging.",
    "version": "3.0.0"
  },
  "plugins": [
    {
      "name": "vibe",
      "source": "./",
      "description": "Persistent project context, auto-generated test harness, and disciplined development workflows for Claude Code.",
      "version": "3.0.0",
      "author": {
        "name": "dukesmith0",
        "url": "https://github.com/dukesmith0"
      },
      "repository": "https://github.com/dukesmith0/vibe-framework",
      "license": "MIT",
      "keywords": [
        "workflow", "context", "planning", "subagents", "testing", "risk-scanning",
        "tdd", "decision-tracking", "autonomous", "code-review", "bugs", "debug"
      ],
      "category": "productivity",
      "tags": ["development", "productivity", "code-quality", "context-management", "testing"]
    }
  ]
}
```

### README.md

Complete rewrite:
- Name: Vibe Framework
- Description: Persistent project context, auto-generated test harness, and disciplined AI development workflows
- Attribution: "Originally created by [Jack Lutz](https://github.com/jwlutz). Forked and extended by [dukesmith0](https://github.com/dukesmith0)."
- Install commands pointing to `dukesmith0/vibe-framework`
- `.vibe/` structure with all 7 files + docs/
- `debug/` test harness explanation
- Command set (13 commands) grouped by use case
- Quick start: `init` → `plan` → `go`
- Why commit `.vibe/` and `debug/` to git

---

## 7. Token Efficiency Strategy

### Targets
- Commands: 60-110 lines (down from 150-230)
- Agents: 15-25 lines (down from 40-60)
- Skills: 30-45 lines
- Total plugin prompt text: ~900-1000 lines (down from ~1400, despite adding features)

### Techniques
1. **Composable pre-compute modules** - 5 modules (_core, _testing, _review, _frontend, _commit), commands compose only what they need
2. **Phase checklists** - numbered steps, not prose paragraphs
3. **Shared context skill** - agent preambles reduced to role-specific only
4. **One example beats three paragraphs** - show format, don't explain it
5. **Rationalization guards** - compact tables, high-impact (superpowers pattern)
6. **Stop points inline** - `[STOP]` / `[STOP IF CRITICAL/HIGH]` markers, not paragraphs
7. **Cross-reference, don't repeat** - "See vibe-context skill for API lookup" not 5 lines of repeated instructions

### What NOT to Cut
- Scene-setting context for subagent dispatch (critical for accuracy)
- Explicit status codes: DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT
- Guards/rationalization blockers (prevent common agent mistakes)
- File format examples (show, don't tell)
- Test framework detection precedence (order matters)
- Subagent fallback pattern ("general-purpose" if unavailable)

---

## 8. File Structure After Rewrite

```
vibe-framework/
├── .claude/
│   └── settings.local.json
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── agents/
│   ├── engineer.md          (~20 lines)
│   ├── explorer.md          (~20 lines)
│   ├── reviewer.md          (~20 lines)
│   └── tester.md            (~20 lines)
├── commands/
│   ├── ask.md               (~22 lines)
│   ├── add.md               (~35 lines)
│   ├── commit.md            (~60 lines)
│   ├── go.md                (~100 lines)
│   ├── help.md              (~25 lines)
│   ├── init.md              (~80 lines)
│   ├── note.md              (~28 lines)
│   ├── oneshot.md           (~95 lines)
│   ├── plan.md              (~70 lines)
│   ├── ralph.md             (~105 lines)
│   ├── refresh.md           (~50 lines)
│   ├── review.md            (~80 lines)
│   └── tdd.md               (~85 lines)
├── skills/
│   ├── vibe-context/
│   │   └── SKILL.md         (~40 lines)
│   └── risk-detection/
│       └── SKILL.md         (~35 lines)
├── .gitignore
├── LICENSE
└── README.md
```

**Removed:** `commands/learn.md`, `commands/log.md`, `commands/risks.md`, `commands/status.md`, `skills/codebase-context/`
**Added:** `commands/plan.md`, `commands/note.md`, `commands/help.md`, `commands/refresh.md`, `commands/add.md`, `skills/vibe-context/`
**Rewritten:** Everything else

---

## 9. Claude Code Platform Integration

Concrete implementations using Claude Code features, not optional suggestions. These are required behaviors.

### Hooks

**PostToolUse hook on Write/Edit:**
When any agent writes or edits a source file, the hook reminds: "File changed: [path]. Update or create corresponding debug/ test if not already done this phase." This ensures no code change goes untested.

**Stop hook (session end):**
Before session ends, check:
- Is `current.md` tracking an active task? If yes, warn: "Active task in progress. current.md state preserved for resume."
- Are there `.vibe/` updates that should be committed? If yes, warn: "Uncommitted .vibe/ changes. Run /vibe:commit before ending."

### Background Agents for Ralph Parallelism

When Ralph loop identifies N independent subtasks (no shared dependencies):
1. Dispatch all N engineer subagents in a single message with `run_in_background: true`
2. Main conversation monitors for completion notifications (do NOT poll or sleep)
3. As each engineer completes, dispatch its reviewer + tester pair
4. Track all parallel progress in `current.md` subtask table
5. If subtasks touch overlapping files, use worktree isolation instead of background dispatch

### Git Worktrees for Isolation

When Ralph has subtasks that modify overlapping files:
1. Dispatch engineer with `isolation: "worktree"` - creates a temporary git worktree
2. Engineer works in isolated copy, no conflicts with other subtasks
3. After review passes, changes are merged back from the worktree branch
4. Worktree auto-cleaned if no changes made
5. Also used for `/vibe:go` when exploring risky changes that might need rollback

### TodoWrite for Real-Time Progress

Every workflow command (go, tdd, oneshot, ralph, review, commit) creates TodoWrite items for its phases:
```
Example for /vibe:go:
1. [completed] Pre-compute environment
2. [completed] Explorer analyzed codebase
3. [completed] Plan approved by user
4. [in_progress] Engineer implementing changes
5. [pending] Review + test + risk scan
6. [pending] Update .vibe/ files
7. [pending] Verify success criteria
```
Rules:
- Create todos at command start, one per phase
- Mark `in_progress` before starting each phase (exactly one at a time)
- Mark `completed` immediately after each phase finishes
- If a phase fails, keep it as `in_progress` and add a new todo for the fix

### Plan Mode for /vibe:plan

`/vibe:plan` uses `EnterPlanMode` at the start of brainstorming:
1. Enter plan mode (prevents accidental code writes during design phase)
2. Ask clarifying questions one at a time (prefer multiple choice when possible)
3. Explore approaches, present trade-offs
4. Present plan with success criteria
5. `ExitPlanMode` when user chooses "execute now" or "save for later"
6. If "execute now", transition to `/vibe:go` (which operates in normal mode, not plan mode)

### Context7 + WebSearch Fallback Chain

Implemented in the `vibe-context` skill, used by ALL agents:

```
1. Is Context7 MCP available?
   YES → resolve-library-id → query-docs → use result
   NO  → step 2
2. Is WebSearch available?
   YES → search "[library] [api] documentation" → read relevant results
   NO  → step 3
3. Read source code comments, type definitions, README
   Found → use what's available, flag low confidence
   NOT FOUND → report NEEDS_CONTEXT to controller
```

This chain runs BEFORE any agent uses an unfamiliar API. Never guess. Always verify.
Agents must explicitly state which step of the chain provided their information.

---

## 10. Additional Scenarios & Edge Cases

**Multi-language debug/ structure:**
When a project uses multiple languages (e.g., Python backend + TypeScript frontend), `debug/` creates per-language subdirectories:
```
debug/
├── python/
│   ├── conftest.py
│   ├── test_api_routes.py
│   └── test_auth.py
├── typescript/
│   ├── setup.ts
│   ├── test_dashboard.spec.ts
│   └── test_components.spec.ts
└── README.md          # Which frameworks, how to run each
```
Each subdirectory uses the project's existing framework for that language. Tester runs both suites.

**Running debug/ tests standalone:**
Users can run `debug/` tests directly without a vibe command using the project's test runner:
- Python: `pytest debug/python/`
- JS/TS: `npx vitest debug/typescript/` or `npx jest debug/typescript/`
- Go: `go test ./debug/...`
No vibe command needed. Tests are standard test files in standard frameworks.

**Risk acknowledgment:**
Users can mark risks as accepted via `/vibe:note decision "Accepting [LOW] unused imports in helpers.ts - cleanup deferred"`. This records in `decisions.md` and the risk stays in `risks.md` but is tagged `[ACCEPTED]` so review commands don't re-flag it.

**Ralph self-review before external review:**
Each Ralph subtask engineer performs a self-review checklist before reporting DONE:
1. Did I miss any requirements from the subtask spec?
2. Do names and patterns match the project conventions?
3. Are debug/ tests comprehensive for what I changed?
4. Would I flag anything if I were reviewing this?
If self-review finds issues, engineer fixes before reporting. This reduces review iteration count.

**Pause/resume mid-workflow:**
If user needs to pause mid-`/vibe:go` (not abandon, just pause):
- `current.md` already tracks progress and phase
- User can close the session
- Next session: `/vibe:go` detects active task in `current.md`, offers to resume from last phase
- All context is in `.vibe/` files, so resume works across sessions

**What changed since last session:**
`/vibe:ask "what changed since last session"` reads `current.md`, `decisions.md` (recent entries), and `git log` to provide a summary. No dedicated command needed since `/vibe:ask` searches both `.vibe/` and the codebase.

---

## 11. Success Criteria

### Core Functionality
- [ ] All 13 commands functional
- [ ] `.vibe/` structure with 7 files + docs/ created correctly by init
- [ ] `debug/` test harness auto-generated by init, maintained by all code-generating workflows
- [ ] `debug/` tests run automatically during go, oneshot, ralph, review, commit, refresh
- [ ] `debug/` tests gate commits - failures block commit
- [ ] Init handles: new projects (empty), existing .vibe/ (incremental), existing debug/ (merge prompt)
- [ ] Init checks git/GitHub status and offers to create repo if missing
- [ ] Bugs and risks use 4-level impact system: CRITICAL, HIGH, MEDIUM, LOW

### Auto-Maintenance
- [ ] Workflows update .vibe/ files without manual intervention
- [ ] Deleted file reconciliation: refresh removes stale entries from understanding.md and debug/
- [ ] New entry point detection: refresh finds and indexes new entry points
- [ ] Dependency change detection: refresh updates Stack section when manifests change
- [ ] Test framework change detection: refresh regenerates debug/ if framework switches
- [ ] Risk baseline updates after commit and review, not mid-workflow

### Plan & Task Management
- [ ] Plan lifecycle works: create → active → complete/defer/replace with proper archival format
- [ ] `current.md` tracks active task state, ralph loop state with subtask table
- [ ] Command conflict prevention: commands check current.md before starting, offer resume/abandon/new
- [ ] current.md lifecycle: created on start, updated per phase, cleared on complete

### Quality Gates
- [ ] Two-stage review enforced: spec compliance first, then code quality
- [ ] Five-part review in /vibe:review: code quality, bugs, functionality, risks, simplification
- [ ] /vibe:go runs comprehensive review automatically after implementation
- [ ] Review verdict gates: CRITICAL/HIGH blocks, MEDIUM requires user approval, LOW reported only
- [ ] 3-fix rule enforced: engineer escalates after 3 failed attempts
- [ ] Ralph self-review before external review
- [ ] Risk baseline comparison: commands report specific added/resolved risks per impact level

### Context & Intelligence
- [ ] Context7 integration: agents use resolve-library-id -> query-docs when available, WebSearch fallback
- [ ] Agents use shared vibe-context skill, no repeated preambles
- [ ] Subagent fallback to "general-purpose" with manual context pasting when vibe agents unavailable
- [ ] Parallel dispatch: reviewer + tester always in single message
- [ ] decisions.md captures: user decisions [USER], Claude decisions [CLAUDE], assumptions, learned lessons
- [ ] Assumption lifecycle: ACTIVE -> VERIFIED or CORRECTED
- [ ] Decision auto-detection rules defined per command
- [ ] Note command auto-detects type from content

### Claude Code Platform
- [ ] PostToolUse hook on Write/Edit reminds agents to update debug/ tests
- [ ] Stop hook warns about active tasks and uncommitted .vibe/ changes
- [ ] TodoWrite creates phase items at command start, updates in real-time
- [ ] Plan mode (EnterPlanMode) used during /vibe:plan brainstorming
- [ ] Background agents (`run_in_background`) for Ralph independent subtasks
- [ ] Git worktree isolation for Ralph overlapping subtasks
- [ ] Context7 -> WebSearch -> source code fallback chain in vibe-context skill
- [ ] Agents state which fallback step provided their API info
- [ ] Test framework prompt when none detected: recommend best, user chooses

### Integration
- [ ] Playwright: detected and used for frontend changes, skipped for backend-only, dev server auto-start with timeout
- [ ] Pre-compute modules are composable, commands use only the modules they need
- [ ] No-git projects: commands work with timestamps, push/PR features disabled gracefully
- [ ] Add command handles heavy formats, large file prompting, folder imports, path validation
- [ ] Multi-language debug/: per-language subdirectories with appropriate framework each
- [ ] debug/ tests runnable standalone via standard test runners (no vibe dependency)
- [ ] Risk acknowledgment via /vibe:note + [ACCEPTED] tag in risks.md
- [ ] Pause/resume across sessions via current.md state persistence
- [ ] Plan->go transition: /vibe:plan "execute now" automatically enters /vibe:go (skip re-planning)
- [ ] Bug statuses: Open, Deferred (with reason), Resolved (with regression test ref)

### Plugin & Branding
- [ ] Token budget: total plugin under 1000 lines
- [ ] Risk detection: language-agnostic + JS/TS + Python patterns
- [ ] README, plugin.json, marketplace.json rebranded with attribution to Jack Lutz
- [ ] No Co-Authored-By lines in commits
- [ ] Error fallbacks defined for every failure mode
