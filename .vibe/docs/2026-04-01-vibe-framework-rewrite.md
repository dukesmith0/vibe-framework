# Vibe Framework v3.0 Rewrite - Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ground-up rewrite of the vibe Claude Code plugin with persistent `.vibe/` context, auto-generated `debug/` test harness, and disciplined subagent-orchestrated workflows.

**Architecture:** Claude Code plugin with 13 slash commands, 4 specialized subagents, 2 reactive skills, and a shared context loading system. All commands read/write `.vibe/` files for persistent project context. Commands use composable pre-compute bash modules. Agents are concise (15-25 lines) with shared behavior in the `vibe-context` skill.

**Tech Stack:** Claude Code plugin system (markdown frontmatter + prompt body), bash for pre-compute, git/gh CLI for version control integration.

**Spec:** `docs/superpowers/specs/2026-04-01-vibe-framework-redesign.md`

---

## Task 1: Cleanup and Plugin Metadata

**Files:**
- Remove: `commands/learn.md`, `commands/log.md`, `commands/risks.md`, `commands/status.md`, `skills/codebase-context/SKILL.md`
- Rewrite: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.claude/settings.local.json`, `.gitignore`

- [ ] **Step 1: Remove deprecated files**

```bash
rm commands/learn.md commands/log.md commands/risks.md commands/status.md
rm -rf skills/codebase-context/
```

- [ ] **Step 2: Write plugin.json**

Write to `.claude-plugin/plugin.json`:
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

- [ ] **Step 3: Write marketplace.json**

Write to `.claude-plugin/marketplace.json`:
```json
{
  "name": "vibe-marketplace",
  "description": "Persistent project context, auto-generated test harness, and disciplined development workflows for Claude Code.",
  "owner": {
    "name": "dukesmith0",
    "url": "https://github.com/dukesmith0"
  },
  "metadata": {
    "description": "Persistent .vibe/ project context that survives across sessions. Auto-generated debug/ test harness. Subagent-orchestrated workflows for planning, implementation, review, and testing.",
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

- [ ] **Step 4: Write settings.local.json**

Write to `.claude/settings.local.json`:
```json
{
  "permissions": {
    "allow": [
      "WebSearch"
    ]
  }
}
```

- [ ] **Step 5: Write .gitignore**

Write to `.gitignore`:
```
# OS
.DS_Store
Thumbs.db

# Editor
*.swp
*.swo
*~
.idea/
.vscode/

# Temp
*.tmp
*.bak

# Debug artifacts (track test files, ignore generated snapshots/fixtures)
debug/.snapshots/
debug/temp-fixtures/
```

- [ ] **Step 6: Verify**

```bash
cat .claude-plugin/plugin.json | head -5
cat .claude-plugin/marketplace.json | head -5
ls commands/  # Should NOT contain learn.md, log.md, risks.md, status.md
ls skills/    # Should NOT contain codebase-context/
```

---

## Task 2: Skills - vibe-context and risk-detection

**Files:**
- Create: `skills/vibe-context/SKILL.md`
- Rewrite: `skills/risk-detection/SKILL.md`

- [ ] **Step 1: Create vibe-context skill directory**

```bash
mkdir -p skills/vibe-context
```

- [ ] **Step 2: Write vibe-context SKILL.md**

Write to `skills/vibe-context/SKILL.md`:
```markdown
---
name: vibe-context
description: Auto-loads .vibe/ project context for commands and agents. Triggers on code questions, implementation, reviews, planning. Provides role-based context loading and API lookup chain.
---

## Context Loading

Check if `.vibe/` exists. If not: "No vibe context found. Run /vibe:init to analyze this codebase."

Load files based on caller role:

| Role | Loads |
|------|-------|
| explorer | understanding.md (full), risks.md, bugs.md, plans.md, current.md, docs index |
| engineer | understanding.md (patterns, components, tests), plans.md, current.md, decisions.md (recent 5) |
| reviewer | understanding.md (patterns, tests), risks.md (baseline), bugs.md |
| tester | understanding.md (tests section only), bugs.md (for regression checks) |
| general | understanding.md, plans.md, current.md |

Read docs index from understanding.md. If current task relates to an indexed doc, load that doc on-demand from `.vibe/docs/`.

## API Lookup Chain

Before any agent uses an unfamiliar API, run this chain. Never guess. Always verify.

1. Context7 MCP available? YES: `resolve-library-id` then `query-docs`. Use result. State: "Verified via Context7."
2. Context7 unavailable? WebSearch: "[library] [api] documentation". Read results. State: "Verified via WebSearch."
3. No network? Read source code comments, type definitions, README. State: "From source only, low confidence."
4. Nothing found? Report `NEEDS_CONTEXT` to controller. Do not proceed with unverified API.

Agents must state which step provided their information.

## Token Efficiency

When writing to .vibe/ files, follow these rules:
- One entry, max two lines. No prose paragraphs.
- Abbreviate paths when unambiguous.
- Use IDs for cross-reference (#3, #R2) instead of restating.
- Dates as YYYY-MM-DD. Strip filler words.
- Archive resolved entries when >20 accumulate.
```

- [ ] **Step 3: Write risk-detection SKILL.md**

Write to `skills/risk-detection/SKILL.md`:
```markdown
---
name: risk-detection
description: Detects code risks and security issues during review, risk scanning, or code evaluation. Writes findings to risks.md with impact level and file:line references.
---

## Patterns

Scan using Grep with these patterns. Tag each finding with impact level and file:line.

**Language-agnostic (all projects):**
- [CRITICAL] Hardcoded secrets: `(api_key|password|secret|token|private_key)\s*=\s*["'][^"']+["']`
- [HIGH] `.env` files committed to git (check .gitignore)
- [MEDIUM] Functions >100 lines
- [MEDIUM] Nesting >4 levels deep
- [MEDIUM] TODO/FIXME/HACK/XXX clusters (>3 in one file)
- [LOW] Files >500 lines

**JavaScript/TypeScript:**
- [CRITICAL] `eval()`, `new Function()`
- [HIGH] `innerHTML`, `dangerouslySetInnerHTML`
- [HIGH] Unvalidated `req.params`, `req.query`, `req.body`
- [LOW] `console.log` in production code (not in debug/ or test files)

**Python:**
- [CRITICAL] `exec()`, `eval()`
- [HIGH] `cursor.execute` with string formatting (SQL injection)
- [HIGH] Bare `except:` (swallows all errors)
- [LOW] `print()` in production code (not in debug/ or test files)

## Output

Write findings to `risks.md` using sequential IDs (#R1, #R2...). One line per risk, max two lines.
Format: `#RN [IMPACT] Description. file:line (found DATE)`

Compare against existing risks.md baseline. Report delta: added (list), resolved (list), net change per impact level.
```

- [ ] **Step 4: Verify skill files**

```bash
cat skills/vibe-context/SKILL.md | head -3
cat skills/risk-detection/SKILL.md | head -3
```

---

## Task 3: Agents - Explorer, Engineer, Reviewer, Tester

**Files:**
- Rewrite: `agents/explorer.md`, `agents/engineer.md`, `agents/reviewer.md`, `agents/tester.md`

- [ ] **Step 1: Write explorer agent**

Write to `agents/explorer.md`:
```markdown
---
name: explorer
description: Explores codebases to map architecture, patterns, and concerns. Use when you need to understand how code works before making changes.
subagent_type: vibe:explorer
tools: Read, Glob, Grep, Bash, WebSearch, WebFetch, LSP
---

## Role

Investigates codebases to map architecture, patterns, and concerns. Goes broad first (structure), then deep (specific files). Verifies framework behavior via docs before reporting.

## Process

1. Read `.vibe/` context via vibe-context skill (role: explorer)
2. Scan project structure with Glob, identify architecture type and organization
3. Read key files to understand components, data flow, dependencies
4. Identify patterns (naming, error handling, test conventions, config)
5. Verify framework/library behavior via API lookup chain (Context7 -> WebSearch -> source)
6. Flag concerns: risks, gaps, assumptions, anything that could cause problems

## Output

Report with file:line references throughout:
- Key files with purposes
- How components connect
- Patterns observed
- Frameworks/libraries identified (verified, not assumed)
- Concerns and risks noticed

## Guards

- Don't skim. Read files completely before reporting.
- Don't assume framework behavior. Verify via Context7/docs.
- Don't omit concerns to seem positive. Flag everything.
```

- [ ] **Step 2: Write engineer agent**

Write to `agents/engineer.md`:
```markdown
---
name: engineer
description: Implements code changes following project patterns exactly. Use when code needs to be written or modified.
subagent_type: vibe:engineer
tools: Read, Glob, Grep, Bash, WebSearch, WebFetch, LSP
---

## Role

Implements code changes following project patterns exactly. Research-first: reads .vibe/ context, verifies APIs via lookup chain, then codes. Generates/updates debug/ tests for all changes.

## Process

1. Read `.vibe/` context via vibe-context skill (role: engineer)
2. Verify unfamiliar APIs via lookup chain (Context7 -> WebSearch -> source). State which step provided info.
3. Implement matching existing code style, patterns, and conventions exactly
4. Generate/update `debug/` tests for changed code
5. Run tests to verify changes work
6. Self-review before reporting: requirements met? patterns matched? tests comprehensive?
7. List all files changed with reasoning

## Output

Status: `DONE` | `DONE_WITH_CONCERNS` | `BLOCKED` | `NEEDS_CONTEXT`
- DONE: files changed with reasoning
- DONE_WITH_CONCERNS: files changed + specific doubts listed
- BLOCKED: what failed, why it's architectural not a bug
- NEEDS_CONTEXT: what information is missing

Record assumptions as `[CLAUDE]` entries in decisions.md.

## Guards

- Don't guess APIs. Look them up before using.
- Don't skip existing patterns. Match them exactly.
- Don't add unrequested features or "improvements."
- **3-fix rule:** If 3 fix attempts fail, report BLOCKED. It's architectural, not a bug.
```

- [ ] **Step 3: Write reviewer agent**

Write to `agents/reviewer.md`:
```markdown
---
name: reviewer
description: Adversarial code review for bugs, security, functionality, and style. Use after code is written or modified.
subagent_type: vibe:reviewer
tools: Read, Glob, Grep, Bash, WebSearch, WebFetch, LSP
---

## Role

Adversarial code review. Two-stage: spec compliance first, then code quality. Verifies library API correctness via Context7/docs. Covers five review areas: code quality, bugs, functionality, risks, simplification.

## Process

1. Read `.vibe/` context via vibe-context skill (role: reviewer)
2. **Stage 1 - Spec compliance:** Do changes match the plan/intent from plans.md?
3. **Stage 2 - Code quality:** Check correctness, security, patterns, style, error handling
4. **Bug review:** Logic errors, edge cases, race conditions, null handling, error propagation
5. **Functionality review:** Does it do what was intended? All code paths reachable and tested?
6. **Risk scan:** Compare against risks.md baseline. Report delta per impact level.
7. Verify library APIs used correctly via lookup chain
8. Propose simplifications for recently changed code only (before/after snippets)

## Output

```
[CRITICAL/HIGH/MEDIUM/LOW] file:line: issue - suggestion
...
Verdict: APPROVE | NEEDS FIXES
```

Write bugs to bugs.md with next ID and impact. Write risks to risks.md with next ID.

## Guards

- Don't rubber-stamp. If uncertain, it's a WARNING not a pass.
- Don't suggest style changes that contradict project patterns.
- Don't skip edge cases because the happy path works.
```

- [ ] **Step 4: Write tester agent**

Write to `agents/tester.md`:
```markdown
---
name: tester
description: Runs project tests, debug/ suite, and linters. Reports results with failure details. Use after code is written or modified.
subagent_type: vibe:tester
tools: Read, Glob, Grep, Bash, WebSearch, WebFetch
---

## Role

Runs project tests + debug/ suite + linters. Reports results with exact failure details and file:line references.

## Process

1. Read `.vibe/` context via vibe-context skill (role: tester)
2. Detect test framework (precedence: pytest.ini -> pyproject.toml -> package.json -> Makefile -> setup.cfg -> go.mod -> Cargo.toml)
3. Run project tests
4. Run `debug/` tests
5. Run linters if configured
6. Report results with exact failure output

## Output

```
Project Tests: PASS/FAIL (ran/passed/failed)
Debug Tests: PASS/FAIL (ran/passed/failed)
Lint: PASS/FAIL/NOT CONFIGURED
Verdict: READY | NEEDS FIXES
[If failures: exact error output, file:line in source]
```

## Guards

- Don't report PASS if any test was skipped or errored.
- Don't skip linters if they exist.
- Don't mask flaky tests. Report them as WARNING.
```

- [ ] **Step 5: Verify agent files**

```bash
for f in agents/*.md; do echo "=== $f ===" && head -4 "$f"; done
```

---

## Task 4: Simple Commands - help, ask, note, add

**Files:**
- Create: `commands/help.md`, `commands/note.md`, `commands/add.md`
- Rewrite: `commands/ask.md`

- [ ] **Step 1: Write help command**

Write to `commands/help.md`:
```markdown
---
description: Show available vibe commands and suggest next action based on project state
---

Check project state:
```bash
test -d .vibe && echo "vibe:initialized" || echo "vibe:none"
test -f .vibe/current.md && head -1 .vibe/current.md
test -f .vibe/plans.md && head -1 .vibe/plans.md
git status -s 2>/dev/null | head -1
```

Show commands grouped by use case:

**Getting started:** `/vibe:init` (analyze codebase), `/vibe:help` (this)
**Planning:** `/vibe:plan <goal>` (brainstorm + plan), `/vibe:ask <question>` (query context)
**Building:** `/vibe:go <task>` (guided workflow), `/vibe:tdd <feature>` (test-first), `/vibe:oneshot <task>` (minimal stops), `/vibe:ralph <plan>` (autonomous multi-task)
**Maintaining:** `/vibe:review` (adversarial review), `/vibe:commit` (smart commit), `/vibe:refresh` (sync after manual changes), `/vibe:note <content>` (quick entry), `/vibe:add <path>` (import docs)

Suggest next action based on state:
- No `.vibe/`? -> "Run `/vibe:init` to analyze your codebase."
- `current.md` shows active task? -> "Active task in progress. Run `/vibe:go` to resume."
- `plans.md` has a plan? -> "You have an active plan. Run `/vibe:go` to continue."
- Git changes staged? -> "Changes ready. Run `/vibe:commit` to commit."
- No git? -> "Note: No git repo detected. Most commands work, but commit/push disabled. Run `/vibe:init` to set up."
```

- [ ] **Step 2: Write ask command**

Write to `commands/ask.md`:
```markdown
---
description: Answer questions about this codebase using .vibe/ context and code search
argument-hint: <question>
---

1. Read `.vibe/` context via vibe-context skill (role: general). If no `.vibe/`: "No vibe context found. Run /vibe:init to analyze this codebase."
2. Check docs index in understanding.md. If question relates to an indexed doc, load it from `.vibe/docs/`.
3. Search actual codebase with Grep/Read for specific details.
4. Answer with file:line citations and code snippets where helpful.

Keep answers concise. Reference specific files and line numbers. If the answer reveals a pattern or decision worth recording, suggest: "Record this with `/vibe:note decision <summary>`."
```

- [ ] **Step 3: Write note command**

Write to `commands/note.md`:
```markdown
---
description: Quick entry to any .vibe/ file - auto-detects type or accepts explicit type
argument-hint: [bug|risk|decision|future] <content>
---

1. Check `.vibe/` exists. If not: "No vibe context found. Run /vibe:init first."

2. Parse argument. If first word is a known type (`bug`, `risk`, `decision`, `future`), use it. Otherwise auto-detect from content:
   - Mentions error, crash, broken, fails, wrong -> `bug`
   - Mentions security, vulnerability, hardcoded, exposed, unsafe -> `risk`
   - Mentions decided, chose, use X not Y, always, never -> `decision`
   - Mentions later, someday, should eventually, nice-to-have, future -> `future`
   - If ambiguous, ask user: "Is this a bug, risk, decision, or future plan?"

3. Format and append:
   - `bug` -> `bugs.md` Open section. Read `Next ID` from top, use it, increment counter. Format: `#N [IMPACT] Description. file:line`  Auto-detect impact from keywords or ask user.
   - `risk` -> `risks.md` appropriate impact section. Use next #RN ID. Format: `#RN [IMPACT] Description. file:line (found DATE)`
   - `decision` -> `decisions.md` Technical Decisions section. Format: `DATE [USER] Decision. Why.`
   - `future` -> `future.md`. Format: `- Description. Context and priority.`

4. Confirm: "Added to [file]: [summary]"
```

- [ ] **Step 4: Write add command**

Write to `commands/add.md`:
```markdown
---
description: Import file or folder to .vibe/docs/ for reference - converts heavy formats, indexes automatically
argument-hint: <path>
---

1. Check `.vibe/` exists. If not: "No vibe context found. Run /vibe:init first."

2. Validate path exists (file or folder). If not found, report error and suggest similar paths if possible.

3. Copy to `.vibe/docs/`, preserving folder structure for directories.

4. For each file:
   - Text files (`.md`, `.txt`, `.json`, `.yaml`, `.csv`, etc.) -> copy as-is
   - Heavy formats (PDF, images, `.docx`, `.pptx`, etc.) -> copy original + generate companion `.md` with extracted text content, same filename
   - Large files (>10 pages or >50KB text): ask user "This file is [size]. Import full text or summary only?" Default: under 10 pages -> full, over 10 -> summary. Recommend splitting very large docs into focused sections.

5. Update docs index in `understanding.md`:
   - Add one-line entry per file: `- [Filename](docs/path) - One-line description`
   - Generate the description from the file's first paragraph or title

6. Report: files added, conversions made, index updated.
```

- [ ] **Step 5: Verify command files**

```bash
for f in commands/help.md commands/ask.md commands/note.md commands/add.md; do echo "=== $f ===" && head -3 "$f"; done
```

---

## Task 5: Init Command

**Files:**
- Rewrite: `commands/init.md`

- [ ] **Step 1: Write init command**

Write to `commands/init.md`:
```markdown
---
description: Analyze codebase and create .vibe/ context + debug/ test harness
---

Use TodoWrite to track progress through phases.

## Pre-compute

```bash
# Core
git rev-parse --git-dir 2>/dev/null && echo "git:yes" || echo "git:none"
git remote -v 2>/dev/null | head -2
test -d .vibe && echo "vibe:exists" || echo "vibe:none"
test -d debug && echo "debug:exists" || echo "debug:none"

# Test framework detection
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || \
(grep -q "^test:" Makefile 2>/dev/null && echo "test:make") || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"
```

## Phases

**Phase 1 - Version control check:**
- If no git: ask "Initialize git repository? [yes/no]" -> if yes, `git init`
- If git but no remote: ask "Create a GitHub repository? [yes/no]" -> if yes, ask public/private, run `gh repo create`
- If git + remote: confirm and continue

**Phase 2 - .vibe/ structure:**
- If `.vibe/` exists, do incremental update (act like /vibe:refresh). If not, create full structure.
- Create: `.vibe/understanding.md`, `.vibe/current.md`, `.vibe/plans.md`, `.vibe/bugs.md`, `.vibe/risks.md`, `.vibe/future.md`, `.vibe/decisions.md`, `.vibe/docs/`

**Phase 3 - Codebase analysis:**
- Scan structure with Glob (exclude: `node_modules, .git, __pycache__, .venv, venv, build, dist, .pytest_cache, debug/`)
- Top-level first: architecture type, languages, entry points, organization
- Deep dive on key directories: components, patterns, data flow, dependencies
- Write findings to `understanding.md` using token-efficient format (see vibe-context skill rules)

**Phase 4 - Risk scan:**
- Run risk-detection skill patterns against codebase
- Write findings to `risks.md` with sequential IDs and impact levels
- Format: `#RN [IMPACT] Description. file:line (found DATE)`

**Phase 5 - Docs indexing:**
- If `.vibe/docs/` contains files, generate docs index in `understanding.md`
- For heavy formats (PDF, images), generate companion `.md` files

**Phase 6 - Test framework:**
- If test framework detected: use it for `debug/`. Report which framework.
- If no framework detected: ask user which to use with recommendation:
  - Python -> recommend pytest
  - JS/TS -> recommend vitest (or jest if already in deps)
  - Go -> `go test` (built-in)
  - Rust -> `cargo test` (built-in)
  - Multi-language -> ask per language
- Record choice in `understanding.md` Tests section

**Phase 7 - debug/ test harness:**
- If `debug/` exists with user-written tests, ask: "Merge existing tests or start fresh?" Default: merge.
- Create `debug/` structure:
  - Multi-language: per-language subdirectories (`debug/python/`, `debug/typescript/`)
  - Single language: flat structure (`debug/conftest.py` or `debug/setup.ts`)
- Generate initial tests: functionality (happy + error paths), edge cases, security checks
- Run `debug/` suite to verify baseline

**Phase 8 - Initialize remaining .vibe/ files:**
- `current.md`: `# No active task`
- `plans.md`: `# Plans` (empty)
- `bugs.md`: `# Bugs\nNext ID: 1\n\n## Open\n\n## Deferred\n\n## Resolved`
- `risks.md`: already populated from Phase 4
- `future.md`: `# Future Plans` (empty)
- `decisions.md`: `# Decisions\n\n## Technical Decisions\n\n## Assumptions\n\n## Learned Lessons\n\n## Plan Archive`
- For empty/new projects: `understanding.md` = "New project, no code analyzed yet."

**Phase 9 - Output summary:**
Report: git status (repo/remote), component count, risk counts by impact level, patterns identified, test count, framework used.
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/init.md
wc -l commands/init.md  # Target: 70-90 lines
```

---

## Task 6: Refresh Command

**Files:**
- Create: `commands/refresh.md`

- [ ] **Step 1: Write refresh command**

Write to `commands/refresh.md`:
```markdown
---
description: Sync .vibe/ and debug/ after manual code changes outside vibe commands
---

Use TodoWrite to track progress.

For use after manual code changes, dependency updates, or file reorganization.

## Pre-compute

```bash
# Core
git status -s 2>/dev/null | head -10
ls .vibe/ 2>/dev/null
head -5 .vibe/understanding.md 2>/dev/null

# Test framework detection
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"

# Playwright detection
test -f playwright.config.ts && echo "playwright:yes" || test -f playwright.config.js && echo "playwright:yes" || \
(grep -q playwright package.json 2>/dev/null && echo "playwright:yes") || echo "playwright:no"
```

If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phases

**Phase 1 - Reconcile understanding.md:**
- Check all components listed against filesystem. Remove entries for deleted files.
- Scan for new entry points (main.*, index.*, server.*, app.*, routes/*, handlers/*). Add to understanding.md.
- Check package manifests (package.json, pyproject.toml, go.mod, Cargo.toml) against Stack section. Update if deps changed.
- Check config files (.env.example, docker-compose, config/) for changes. Update if needed.
- If >30% of listed files changed, do full re-scan (like init Phase 3). Otherwise delta-update.

**Phase 2 - Reconcile debug/:**
- Validate all test imports. Remove tests that reference deleted source files.
- Check if test framework changed (different from what debug/ uses). If so, regenerate debug/ with new framework. Report the switch.
- Scan for untested new code. Generate missing tests.

**Phase 3 - Docs check:**
- Scan `.vibe/docs/` for new/changed files. Update docs index. Generate `.md` companions for new heavy formats.

**Phase 4 - Risk re-scan:**
- Run risk-detection skill patterns. Update `risks.md` with baseline comparison.
- Move resolved risks (file:line no longer matches pattern) to Resolved section.

**Phase 5 - Run debug/ suite:**
- Run full debug/ test suite to verify current state.

**Phase 6 - Report:**
What changed in understanding, files removed, tests added/removed, new risks found, risk delta, test results.
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/refresh.md
wc -l commands/refresh.md  # Target: 45-55 lines
```

---

## Task 7: Plan Command

**Files:**
- Create: `commands/plan.md`

- [ ] **Step 1: Write plan command**

Write to `commands/plan.md`:
```markdown
---
description: Brainstorm, ask questions, create an implementation plan - then execute or save for later
argument-hint: <goal>
---

Use TodoWrite to track progress. Use EnterPlanMode at start to prevent accidental code changes during design.

## Pre-compute

```bash
git status -s 2>/dev/null | head -5
ls .vibe/ 2>/dev/null
head -5 .vibe/plans.md 2>/dev/null
head -1 .vibe/current.md 2>/dev/null
cat .vibe/future.md 2>/dev/null
```

If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phases

**Phase 1 - Enter plan mode:**
- Use EnterPlanMode to prevent code writes during brainstorming.
- Read `understanding.md` for project context.
- Read `future.md` to check if goal matches an existing future plan. If match found, surface it: "Found a related plan in future.md: [summary]. Build on this?"
- Read `decisions.md` for relevant prior decisions.

**Phase 2 - Clarifying questions:**
- Ask questions one at a time. Prefer multiple choice when possible.
- Focus on: purpose, constraints, success criteria, scope, integration points.
- Keep going until you understand the goal well enough to propose approaches.
- Do NOT ask more than 5 questions unless the goal is genuinely ambiguous.

**Phase 3 - Propose approaches:**
- Present 2-3 different approaches with trade-offs.
- Lead with your recommendation and explain why.
- Each approach: name, description, pros, cons, estimated complexity.

**Phase 4 - Present plan:**
- After user chooses approach, present the plan with:
  - Goal (one sentence)
  - Approach (2-3 sentences)
  - Success criteria as `- [ ]` checkboxes (measurable, specific)
  - Tasks as `- [ ]` checkboxes (actionable steps)
- Record user's chosen approach as `[USER]` decision in `decisions.md`.

**Phase 5 - Execute or defer:**
- ExitPlanMode.
- [STOP] Ask: "Execute now or save for later?"
- **Execute now:**
  - Check `current.md`. If active task, ask: "Active task found: [description]. Resume, abandon, or start new?"
  - Archive existing plan if any (to `decisions.md` Plan Archive with outcome).
  - Write plan to `plans.md`. Update `current.md`.
  - Automatically begin `/vibe:go` workflow, skipping go's explore+plan phases (jump to go phase 6: implement).
- **Save for later:**
  - Append to `future.md` with one-line summary.
  - Return to user prompt.
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/plan.md
wc -l commands/plan.md  # Target: 60-80 lines
```

---

## Task 8: Go Command

**Files:**
- Rewrite: `commands/go.md`

- [ ] **Step 1: Write go command**

Write to `commands/go.md`:
```markdown
---
description: Full development workflow - explore, plan, implement, review, verify
argument-hint: <task>
---

Use TodoWrite to track progress through all phases.

## Pre-compute

```bash
# Core
git status -s 2>/dev/null | head -5
git branch --show-current 2>/dev/null
ls .vibe/ 2>/dev/null
head -5 .vibe/plans.md 2>/dev/null
head -1 .vibe/current.md 2>/dev/null

# Testing
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
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
Run bash block above. Read `.vibe/` context via vibe-context skill.

## Phase 2 - Entry routing
- If `plans.md` has active plan matching task: offer to continue it (skip to phase 6).
- If `plans.md` has active plan NOT matching task: ask "Continue existing plan or start new?"
- If no plan and no task argument: "No plan or task specified. Run `/vibe:plan <goal>` to create a plan, or provide a task: `/vibe:go <task>`." If user then provides a simple task, suggest `/vibe:oneshot` for faster workflow.
- If task references a bug ID (e.g., "fix #3"): read bug details from `bugs.md`, use as task context, proceed to phase 3.
- If task references a risk ID (e.g., "fix #R2"): read risk details from `risks.md`, use as task context, proceed to phase 3.
- If task provided but no plan: proceed to phase 3.
- Check `current.md`. If active task: "Active task found: [description]. Resume, abandon, or start new?"

## Phase 3 - Explore
Dispatch explorer subagent (`subagent_type="vibe:explorer"` or `"general-purpose"` if unavailable):
- Analyze code relevant to the task
- Identify patterns, risks, test gaps
- Report findings with file:line references

If falling back to general-purpose: paste understanding.md patterns and current plan context directly into the dispatch prompt.

## Phase 4 - Plan
- Present 1-3 approaches with trade-offs based on explorer findings.
- Define success criteria as `- [ ]` checkboxes.
- **[STOP]** Wait for user approval.
- Record chosen approach as `[USER]` decision in `decisions.md`.

## Phase 5 - Write plan
Write approved plan to `plans.md`. Update `current.md` with task state (phase 5/12).

## Phase 6 - Implement
Dispatch engineer subagent (`subagent_type="vibe:engineer"` or `"general-purpose"`):
- Scene-setting: what subsystems affected, which patterns to follow, acceptance criteria
- Follow patterns from `understanding.md`
- Generate/update `debug/` tests for changed code
- Record assumptions as `[CLAUDE]` entries in `decisions.md`

If engineer returns `DONE_WITH_CONCERNS`: read concerns before proceeding.
If engineer returns `BLOCKED`: present to user for discussion. Do not attempt fix #4 (3-fix rule).
If engineer returns `NEEDS_CONTEXT`: provide requested context, re-dispatch.

## Phase 7 - Auto-generate tests
- Update `debug/` tests for all changed code.
- Run full `debug/` suite + project tests.

## Phase 8 - Comprehensive review
Dispatch reviewer + tester subagents **in parallel (single message)**:

**Reviewer** (`subagent_type="vibe:reviewer"` or `"general-purpose"`):
- Stage 1: Spec compliance - does implementation match success criteria from phase 4?
- Stage 2: Code quality - bugs, security, patterns, style (CRITICAL/HIGH/MEDIUM/LOW)
- Bug review: logic errors, edge cases, race conditions, error handling gaps
- Functionality review: does implementation satisfy success criteria?
- Risk scan: compare against `risks.md` baseline, report delta per impact level
- Code simplification: propose simplifications for recently changed code (before/after snippets)

**Tester** (`subagent_type="vibe:tester"` or `"general-purpose"`):
- Run project tests + `debug/` suite + linters
- If Playwright detected AND changes include frontend: run `npx playwright test --reporter=list`. Start dev server if needed.

## Phase 9 - Review gate
- **[STOP IF CRITICAL/HIGH]** Present findings. User approves simplifications individually.
- CRITICAL/HIGH: fix required before proceeding.
- MEDIUM: present to user, user approves to continue or requests fix.
- LOW: report in summary, no gate.

## Phase 10 - Update .vibe/
- `decisions.md`: decisions [USER/CLAUDE] + assumptions from phases 4 and 6
- `bugs.md`: bugs found with next sequential ID and impact level
- `risks.md`: new scan with baseline update. Move resolved risks to Resolved section.
- `current.md`: progress at phase 10
- `understanding.md`: if architecture changed (new top-level dirs, entry points, frameworks, service boundaries)
- `debug/`: regression tests for any bugs resolved during implementation

## Phase 11 - Verify
Check each success criterion from phase 4. If any unmet, go back to phase 6 and fix before completing.

## Phase 12 - Complete
- Clear `current.md` (reset to `# No active task`)
- Archive plan summary to `decisions.md` Plan Archive: `Completed: [title] (DATE) | Done | Summary. Key decisions.`
- Clear `plans.md` (reset to `# Plans`)
- Present summary: what changed, decisions recorded, test results, risk delta.
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/go.md
wc -l commands/go.md  # Target: 90-110 lines
```

---

## Task 9: TDD Command

**Files:**
- Rewrite: `commands/tdd.md`

- [ ] **Step 1: Write tdd command**

Write to `commands/tdd.md`:
```markdown
---
description: Test-driven development - write tests first, lock contract, implement until green
argument-hint: <feature>
---

Use TodoWrite to track progress through all phases.

## Pre-compute

```bash
git status -s 2>/dev/null | head -5
ls .vibe/ 2>/dev/null
head -1 .vibe/current.md 2>/dev/null

# Test framework detection
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"
```

If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phase 1 - Pre-compute
Run bash block. Read `.vibe/` context. Update `current.md` with task state (phase 1/11, TDD mode).

## Phase 2 - Explore
Dispatch explorer subagent (`subagent_type="vibe:explorer"` or `"general-purpose"`):
- Analyze target code for the feature
- Find existing test patterns, test framework APIs
- Identify where tests should live (colocated or `debug/`)
- Verify test framework API via lookup chain (Context7 -> WebSearch)

## Phase 3 - Write tests
Dispatch engineer subagent (`subagent_type="vibe:engineer"` or `"general-purpose"`):
- Write failing tests to `debug/` (or project test location if colocated pattern)
- Tests must be comprehensive: happy path, error paths, edge cases, boundary values
- Record any requirement assumptions in `decisions.md` Assumptions section
- Do NOT write implementation. Only tests.

## Phase 4 - Verify failure
Dispatch tester subagent: run the new tests. Confirm they FAIL for the right reason (missing implementation, not typo/import error). If tests fail for wrong reason, fix tests and re-run.

## Phase 5 - Approve tests
**[STOP]** Present tests to user for approval. Show what each test validates.

## Phase 6 - Commit tests
Commit tests only. This locks the test contract. Tests must NOT be modified after this point.

## Phase 7 - Implement
Dispatch engineer subagent:
- Write minimal code to make all tests pass
- Follow patterns from `understanding.md`
- Record implementation assumptions as `[CLAUDE]` in decisions.md

## Phase 8 - Iterate
Dispatch tester subagent: run all tests (project + debug/ + new TDD tests).
- If passing: proceed to phase 9
- If failing: engineer fixes implementation (NOT tests), tester re-runs
- Max 3 iterations before escalating to user with failure details

## Phase 9 - Update .vibe/
- `current.md`: progress at phase 9
- `decisions.md`: test design assumptions, requirement gaps, implementation choices
- `bugs.md`: any bugs found during testing with ID and impact
- `debug/`: regression tests for any resolved bugs

## Phase 10 - Commit implementation
Commit passing implementation. Present summary: what was tested, what was implemented, iterations needed.
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/tdd.md
wc -l commands/tdd.md  # Target: 70-90 lines
```

---

## Task 10: Oneshot Command

**Files:**
- Rewrite: `commands/oneshot.md`

- [ ] **Step 1: Write oneshot command**

Write to `commands/oneshot.md`:
```markdown
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
Run bash block. Read `.vibe/` context. Check `current.md` for active task (conflict prevention).

## Phase 2 - Clarify
Dispatch explorer subagent (`subagent_type="vibe:explorer"` or `"general-purpose"`):
- Analyze codebase for the task. Present back to user:
  - "What I think you want"
  - "What this involves"
  - "Assumptions I'm making"
  - "Questions before I proceed"

## Phase 3 - Confirm
**[STOP]** User confirms understanding, answers questions, corrects assumptions.
Record user's confirmed understanding as `[USER]` decision in `decisions.md`.

## Phase 4 - Plan + Implement
- Plan internally, write to `plans.md` with success criteria.
- Update `current.md`.
- Dispatch engineer subagent: implement following patterns from understanding.md.
- Record Claude's choices as `[CLAUDE]`, assumptions in decisions.md.

## Phase 5 - Test
- Auto-generate/update `debug/` tests for changed code.
- Run full `debug/` suite + project tests.

## Phase 6 - Review
Dispatch reviewer + tester in parallel (single message):
- Reviewer: two-stage (spec compliance then quality) + bug review + functionality + risk scan
- Tester: project tests + debug/ suite + linters
- If Playwright detected and frontend changes: run Playwright tests

## Phase 7 - Review gate
**[STOP IF CRITICAL/HIGH]** Pause on critical/high issues. User decides fix or continue.

## Phase 8 - Risk scan
Compare against `risks.md` baseline. Report delta per impact level.

## Phase 9 - Simplification
Propose simplifications with before/after snippets for recently changed code only.
**[STOP]** User approves individually. Rules: preserve exact functionality, don't touch untouched code.

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
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/oneshot.md
wc -l commands/oneshot.md  # Target: 80-100 lines
```

---

## Task 11: Ralph Command

**Files:**
- Rewrite: `commands/ralph.md`

- [ ] **Step 1: Write ralph command**

Write to `commands/ralph.md`:
```markdown
---
description: Autonomous multi-task loop - decompose plan into subtasks, implement and review each, report results
argument-hint: <plan>
---

Use TodoWrite to track progress.

## Pre-compute

```bash
# Core
git status -s 2>/dev/null | head -5
git branch --show-current 2>/dev/null
ls .vibe/ 2>/dev/null
head -5 .vibe/plans.md 2>/dev/null
head -1 .vibe/current.md 2>/dev/null

# Testing
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"

# Frontend
test -f playwright.config.ts && echo "playwright:yes" || test -f playwright.config.js && echo "playwright:yes" || \
(grep -q playwright package.json 2>/dev/null && echo "playwright:yes") || echo "playwright:no"
```

If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phase 1 - Pre-compute
Run bash block. Read `.vibe/` context. Check `current.md` for existing ralph state (resume capability).
If active ralph state found: offer to resume from last position.

## Phase 2 - Decompose
Dispatch explorer subagent (`subagent_type="vibe:explorer"` or `"general-purpose"`):
- Break plan into subtasks with:
  - Dependencies (which subtasks must complete first)
  - Confidence level: HIGH (clear spec) / MEDIUM (some ambiguity) / LOW (significant unknowns)
  - Estimated complexity

## Phase 3 - Approve
**[STOP]** Present subtask breakdown as table:
```
| # | Subtask | Confidence | Deps | Complexity |
```
Flag LOW confidence subtasks for user attention. User says "go" to start autonomous loop.

## Phase 4 - Write plan
Write plan to `plans.md` with subtask breakdown. Update `current.md` with ralph state table.

## Phase 5 - The Loop

**Autonomous execution. Once user says "go", do NOT stop until all subtasks are DONE or STUCK. No asking for approval. No pausing for confirmation. Make reasonable assumptions, document them, keep going.**

For each ready subtask (dependencies satisfied):

1. Update `current.md` subtask table: mark IN_PROGRESS
2. **Implement:** dispatch engineer subagent with scene-setting:
   - What subsystems affected
   - What prior subtasks changed and decisions made
   - Acceptance criteria for this subtask
   - Use `run_in_background: true` for independent subtasks
   - Use `isolation: "worktree"` for subtasks touching overlapping files
3. **Self-review:** engineer reviews own work before reporting (requirements met? patterns matched? tests?)
4. **Generate tests:** auto-generate/update `debug/` tests for changed code
5. **Two-stage review:** dispatch reviewer subagent:
   - Stage 1: Spec compliance (matches subtask spec?)
   - Stage 2: Code quality + bugs + security
6. **Test:** dispatch tester in parallel with reviewer (single message). Run project + debug/ tests.
7. **Fix loop:** if review finds issues, same engineer fixes, reviewer re-checks. Max 3 iterations total.
8. **Evaluate:** DONE / DONE_WITH_CONCERNS / STUCK (3 iterations) / BLOCKED (missing info)
9. **Record immediately (not batched):**
   - Claude decisions as `[CLAUDE]` in decisions.md
   - Assumptions in decisions.md Assumptions section
   - Bugs in bugs.md with next ID and impact
   - Risks in risks.md with next ID
10. Update `current.md` subtask table with result and iteration count

Parallel dispatch: if N independent subtasks are ready, dispatch all N engineers in one message with `run_in_background: true`. As each completes, dispatch its reviewer + tester pair.

## Phase 6 - Report
After all subtasks processed, present comprehensive results:
- Subtask status table (DONE/STUCK/BLOCKED with symbols)
- DONE_WITH_CONCERNS items with specific concerns
- Decisions made (already in decisions.md)
- Assumptions made (already in decisions.md)
- Bugs found (already in bugs.md with IDs)
- Risk delta (already in risks.md)
- Things that couldn't be figured out (STUCK/BLOCKED items)

## Phase 7 - Update .vibe/
- Archive plan to `decisions.md` Plan Archive (if all subtasks DONE)
- Update `understanding.md` if architecture changed
- Clear `current.md` if all done. Keep ralph state if STUCK/BLOCKED items remain.

## Phase 8 - Resume
If user responds after report, resume loop for STUCK/BLOCKED items only. Do NOT redo completed subtasks unless explicitly asked. User may provide new context for BLOCKED items.
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/ralph.md
wc -l commands/ralph.md  # Target: 95-110 lines
```

---

## Task 12: Review Command

**Files:**
- Rewrite: `commands/review.md`

- [ ] **Step 1: Write review command**

Write to `commands/review.md`:
```markdown
---
description: Five-part adversarial review - code quality, bugs, functionality, risks, simplification
argument-hint: [scope]
---

Use TodoWrite to track progress.

## Pre-compute

```bash
# Review
git diff --name-only 2>/dev/null | head -20
git diff --stat 2>/dev/null
git diff --staged --name-only 2>/dev/null | head -10

# Core
ls .vibe/ 2>/dev/null
head -3 .vibe/risks.md 2>/dev/null
head -3 .vibe/plans.md 2>/dev/null

# Testing
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
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
Run bash block. Read `.vibe/` context. Identify changed files and scope of review.

## Phase 2 - Code review
Dispatch reviewer subagent (`subagent_type="vibe:reviewer"` or `"general-purpose"`) for adversarial review:

**Stage 1 - Spec compliance:** Do changes match intent/plan from `plans.md`?
**Stage 2 - Code quality:** Patterns, style, naming, duplication, error handling
**Stage 3 - Security:** Input validation, auth, secrets, injection vectors

Output: `[CRITICAL/HIGH/MEDIUM/LOW] file:line: issue - fix`

## Phase 3 - Bug review
Reviewer continues:
- Logic errors, off-by-one, null/undefined handling, race conditions
- Edge cases: empty inputs, boundary values, concurrent access
- Error propagation: exceptions bubble correctly? Error messages useful?
- Cross-reference with `bugs.md`: are any known bugs reintroduced?

## Phase 4 - Functionality review
Reviewer continues:
- Does implementation do what was intended?
- All code paths reachable and tested?
- Integrations with other components work correctly?

## Phase 5 - Test
Dispatch tester in parallel with reviewer (single message, phases 2-5):
- Run project tests + `debug/` suite + linters
- If Playwright detected AND changes include frontend: run `npx playwright test --reporter=list`
  - Check if dev server running (curl localhost:3000/5173). Start with 60s timeout if down.
  - Skip if changes are backend-only or config-only

## Phase 6 - Risk scan
Compare against `risks.md` baseline. Report delta per impact level: added (list), resolved (list), net change.

## Phase 7 - Write findings
- `bugs.md`: bugs found with next sequential ID and impact level
- `risks.md`: risks with baseline update. Move resolved risks to Resolved section.

## Phase 8 - Test gap check
Check `debug/` for gaps: any changed code without corresponding tests? Generate missing tests.

## Phase 9 - Review gate
**[STOP IF CRITICAL/HIGH]** Present all findings with verdict: `APPROVE` or `NEEDS FIXES`

## Phase 10 - Simplification
Propose changes with before/after snippets for recently changed code only.
**[STOP]** User approves individually.
Rules: preserve exact functionality, don't refactor untouched code, don't add features/abstractions/renamed variables.

## Phase 11 - Summary
Impact counts, verdict, risk delta, test results, simplifications applied.

## Phase 12 - Next steps
Based on verdict:
- `APPROVE`: "Ready to commit. Run `/vibe:commit`."
- `NEEDS FIXES` with bugs: "Bugs recorded. Run `/vibe:go fix #N` to address specific bugs, or `/vibe:plan` to plan a broader fix."
- `NEEDS FIXES` with risks: "Risks updated. Run `/vibe:go fix #RN` to address specific risks."
- Mixed: list each with ID. "Run `/vibe:go fix #3` or `/vibe:go fix #R2` individually, or `/vibe:plan fix review findings` to tackle together."
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/review.md
wc -l commands/review.md  # Target: 70-85 lines
```

---

## Task 13: Commit Command

**Files:**
- Rewrite: `commands/commit.md`

- [ ] **Step 1: Write commit command**

Write to `commands/commit.md`:
```markdown
---
description: Smart commit with test gate, .vibe/ sync, and optional PR creation
argument-hint: [message hint]
---

## Pre-compute

```bash
# Commit
git status -s 2>/dev/null
git diff --staged --stat 2>/dev/null
git diff --stat 2>/dev/null
git log --oneline -5 2>/dev/null
git branch --show-current 2>/dev/null
git remote -v 2>/dev/null | head -2

# Testing
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"
```

If no changes staged or unstaged: "No changes to commit." Exit cleanly.

## Phase 1 - Test gate
- Run `debug/` suite. If failures: report and do NOT proceed. Tests must pass before commit.
- Run project tests. Same gate.

## Phase 2 - Analyze
What changed and why. Group into logical commit(s).

## Phase 3 - .vibe/ sync
- Does diff affect architecture/patterns (new top-level dirs, entry points, frameworks)? -> update `understanding.md`
- Does diff resolve any bugs in `bugs.md`? -> move to Resolved section with date, add regression test to `debug/` if not present
- Does diff resolve any risks in `risks.md`? (file:line pattern no longer matches) -> move to Resolved section with date

## Phase 4 - Draft message
- Concise. Follow project style from recent commits (look at `git log --oneline`).
- First line under 72 chars. Add body if "why" isn't obvious.
- Use conventional commits only if existing style uses them.
- **Do NOT add Co-Authored-By lines. User's attribution setting handles this.**

## Phase 5 - Approve
**[STOP]** Present staged changes + draft message for approval. User can edit message.

## Phase 6 - Execute
- Commit with approved message.
- Push to remote. If push fails: check if remote is ahead, pull if needed, retry.
- If pre-commit hook fails: display error. Ask user to fix or run `/vibe:review` to auto-fix. Re-stage, create NEW commit (never amend).

## Phase 7 - PR offer
If on feature branch (not main/master): "Create a pull request? [yes/no]"
If yes: use `gh pr create` with title from commit message and summary of changes.
```

- [ ] **Step 2: Verify**

```bash
head -3 commands/commit.md
wc -l commands/commit.md  # Target: 55-65 lines
```

---

## Task 14: Hooks Configuration

**Files:**
- Create: `hooks/PostToolUse.md`, `hooks/Stop.md`

Note: Claude Code plugin hooks live in a `hooks/` directory at plugin root. They execute automatically on the corresponding event.

- [ ] **Step 1: Create hooks directory**

```bash
mkdir -p hooks
```

- [ ] **Step 2: Write PostToolUse hook**

Write to `hooks/PostToolUse.md`:
```markdown
---
name: vibe-test-reminder
description: Reminds agents to update debug/ tests when source files are modified
hooks:
  - event: PostToolUse
    tools: [Write, Edit]
---

If the file just written/edited is a source file (not in `.vibe/`, `debug/`, `docs/`, or config files):
- Check if a corresponding test exists in `debug/`
- If no test exists: "Reminder: No debug/ test for this file. Generate one before completing this phase."
- If test exists but file changed significantly: "Reminder: debug/ test may need updating for changes in [file]."
```

- [ ] **Step 3: Write Stop hook**

Write to `hooks/Stop.md`:
```markdown
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
```

- [ ] **Step 4: Verify hooks**

```bash
ls hooks/
head -5 hooks/PostToolUse.md
head -5 hooks/Stop.md
```

---

## Task 15: README

**Files:**
- Rewrite: `README.md`

- [ ] **Step 1: Write README**

Write to `README.md`:
```markdown
# Vibe Framework

Persistent project context, auto-generated test harness, and disciplined AI development workflows for Claude Code.

Originally created by [Jack Lutz](https://github.com/jwlutz). Forked and extended by [dukesmith0](https://github.com/dukesmith0).

## Why Use This?

AI coding assistants lose context between sessions and can make changes without understanding the broader system. Vibe Framework fixes this with:

- **Persistent context** in `.vibe/` that survives across sessions
- **Auto-generated tests** in `debug/` that validate everything
- **Structured workflows** that plan before coding, review after, and track every decision
- **Subagent orchestration** with specialized agents for exploration, implementation, review, and testing

## Installation

```bash
claude plugin marketplace add dukesmith0/vibe-framework
claude plugin install vibe@vibe-marketplace
```

### Team Installation

Add to your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "vibe-marketplace": {
      "source": {
        "source": "github",
        "repo": "dukesmith0/vibe-framework"
      }
    }
  },
  "enabledPlugins": {
    "vibe@vibe-marketplace": true
  }
}
```

## Quick Start

```bash
/vibe:init          # Analyze codebase, create .vibe/ and debug/
/vibe:plan Add auth # Brainstorm, plan, then execute or defer
/vibe:go            # Continue with the plan
/vibe:review        # Adversarial review when ready
/vibe:commit        # Smart commit with test gate
```

## Commands

**Getting started:**
| Command | Description |
|---------|-------------|
| `/vibe:init` | Analyze codebase, create `.vibe/` context + `debug/` test harness |
| `/vibe:help` | Show commands and suggest next action |

**Planning:**
| Command | Description |
|---------|-------------|
| `/vibe:plan <goal>` | Brainstorm, ask questions, create plan, execute or defer |
| `/vibe:ask <question>` | Answer questions using context + codebase search |

**Building:**
| Command | Description |
|---------|-------------|
| `/vibe:go <task>` | Full workflow: explore, plan, implement, review, verify |
| `/vibe:tdd <feature>` | Test-driven: write tests first, implement until green |
| `/vibe:oneshot <task>` | Minimal-stop end-to-end workflow |
| `/vibe:ralph <plan>` | Autonomous multi-task loop with parallel execution |

**Maintaining:**
| Command | Description |
|---------|-------------|
| `/vibe:review` | Five-part adversarial review with test gate |
| `/vibe:commit` | Smart commit with test gate and .vibe/ sync |
| `/vibe:refresh` | Sync .vibe/ after manual code changes |
| `/vibe:note <content>` | Quick entry to bugs, risks, decisions, or future plans |
| `/vibe:add <path>` | Import reference docs to .vibe/docs/ |

## What Gets Created

```
.vibe/
├── understanding.md   # Architecture, stack, components, patterns
├── current.md         # Active task state and progress
├── plans.md           # Current plan with success criteria
├── bugs.md            # Tracked bugs with IDs and impact levels
├── risks.md           # Risks with IDs, impact, and baseline tracking
├── future.md          # Backlog and deferred plans
├── decisions.md       # Decisions, assumptions, and learned lessons
└── docs/              # Reference materials

debug/
├── conftest.py        # Test fixtures (framework-dependent)
├── test_*.py          # Auto-generated comprehensive tests
└── ...                # Mirrors source structure
```

**Commit both folders.** They give Claude persistent context and comprehensive test coverage across sessions.

## The Primary Loop

```
/vibe:init                    # Once per project
  -> /vibe:plan <goal>        # Brainstorm and plan
    -> /vibe:go               # Implement the plan
      -> /vibe:review         # Review changes
        -> /vibe:commit       # Ship it
  -> /vibe:plan <next goal>   # Repeat
```

Every step auto-updates `.vibe/` files and `debug/` tests. Bugs and risks get IDs (#3, #R2) so you can fix them directly: `/vibe:go fix #3`.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- `gh` CLI (optional, for GitHub repo creation and PR workflows)

## License

MIT - see [LICENSE](LICENSE)
```

- [ ] **Step 2: Verify**

```bash
head -5 README.md
wc -l README.md
```

---

## Final Verification

- [ ] **Step 1: Check all files exist**

```bash
echo "=== Plugin ===" && ls .claude-plugin/
echo "=== Settings ===" && ls .claude/
echo "=== Agents ===" && ls agents/
echo "=== Commands ===" && ls commands/
echo "=== Skills ===" && ls skills/*/
echo "=== Hooks ===" && ls hooks/
echo "=== Root ===" && ls README.md LICENSE .gitignore
```

Expected commands (13): `add.md ask.md commit.md go.md help.md init.md note.md oneshot.md plan.md ralph.md refresh.md review.md tdd.md`

Expected agents (4): `engineer.md explorer.md reviewer.md tester.md`

Expected skills (2): `vibe-context/SKILL.md risk-detection/SKILL.md`

- [ ] **Step 2: Check no deprecated files remain**

```bash
test -f commands/learn.md && echo "FAIL: learn.md exists" || echo "OK: learn.md removed"
test -f commands/log.md && echo "FAIL: log.md exists" || echo "OK: log.md removed"
test -f commands/risks.md && echo "FAIL: risks.md exists" || echo "OK: risks.md removed"
test -f commands/status.md && echo "FAIL: status.md exists" || echo "OK: status.md removed"
test -d skills/codebase-context && echo "FAIL: codebase-context exists" || echo "OK: codebase-context removed"
```

- [ ] **Step 3: Line count audit**

```bash
for f in commands/*.md; do echo "$(wc -l < "$f") $f"; done | sort -n
for f in agents/*.md; do echo "$(wc -l < "$f") $f"; done | sort -n
for f in skills/*/SKILL.md; do echo "$(wc -l < "$f") $f"; done | sort -n
echo "Total:" && find commands agents skills -name "*.md" -exec cat {} + | wc -l
```

Target: total plugin under 1000 lines.
