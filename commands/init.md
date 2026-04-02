---
description: Analyze codebase and create .vibe/ context + debug/ test harness
---

Use TodoWrite to track progress through phases.

## Pre-compute

```bash
# Version control
git rev-parse --git-dir 2>/dev/null && echo "git:yes" || echo "git:none"
git remote -v 2>/dev/null | head -2

# Existing state
test -d .vibe && echo "vibe:exists" || echo "vibe:none"
test -d debug && echo "debug:exists" || echo "debug:none"

# Test framework detection (order matters - first match wins)
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(test -f vitest.config.ts -o -f vitest.config.js && echo "test:vitest") || \
(test -f jest.config.ts -o -f jest.config.js && echo "test:jest") || \
(node -e "const p=require('./package.json');if(p.scripts?.test){console.log('test:'+p.scripts.test);process.exit(0)}else{process.exit(1)}" 2>/dev/null) || \
(grep -q "^test:" Makefile 2>/dev/null && echo "test:make") || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"

# Language detection
find . -maxdepth 3 -name "*.py" -not -path "./.git/*" -not -path "./.vibe/*" 2>/dev/null | head -1 | grep -q . && echo "lang:python"
find . -maxdepth 3 -name "*.ts" -o -name "*.tsx" 2>/dev/null | head -1 | grep -q . && echo "lang:typescript"
find . -maxdepth 3 -name "*.js" -o -name "*.jsx" 2>/dev/null | head -1 | grep -q . && echo "lang:javascript"
find . -maxdepth 3 -name "*.go" 2>/dev/null | head -1 | grep -q . && echo "lang:go"
find . -maxdepth 3 -name "*.rs" 2>/dev/null | head -1 | grep -q . && echo "lang:rust"
```

## Phase 1 - Version control

Check git status and offer setup if needed:

- If no git repo: ask "Initialize git repository? [yes/no]". If yes: `git init`
- If git but no remote: ask "Create a GitHub repository? [yes/no]". If yes: ask public/private, run `gh repo create --source=. --push` with appropriate flags
- If git + remote: note repo info and continue

## Phase 2 - .vibe/ structure

If `.vibe/` exists: do incremental update (act like `/vibe:refresh`). Report: "Existing .vibe/ found. Updating."
If not: create full structure:

```bash
mkdir -p .vibe/docs
```

Create these files (content below):
- `.vibe/understanding.md` - populated in Phase 3
- `.vibe/current.md` - `# No active task`
- `.vibe/bugs.md` - see template below
- `.vibe/risks.md` - populated in Phase 4
- `.vibe/future.md` - `# Future Plans`
- `.vibe/decisions.md` - see template below

**bugs.md template:**
```markdown
# Bugs
Next ID: 1

## Open

## Deferred

## Resolved
```

**decisions.md template:**
```markdown
# Decisions

## Technical Decisions

## Assumptions

## Learned Lessons

## Plan Archive
```

## Phase 3 - Codebase analysis

Scan codebase and write findings to `understanding.md`. Use token-efficient format.

1. Scan structure with Glob (exclude: `node_modules, .git, __pycache__, .venv, venv, build, dist, .pytest_cache, debug/, .vibe/`)
2. **Top-level first:** architecture type (monorepo, single-app, library, CLI), languages, entry points, high-level organization
3. **Deep dive on key directories:** read main files, understand components, data flow, dependencies
4. **Patterns:** naming conventions, error handling, test conventions, config patterns
5. Write to `understanding.md` in this format:

```markdown
# project-name
Last: YYYY-MM-DD | N files | Languages

## Stack
One line per technology. Framework + version + purpose.

## Architecture
2-3 sentences. Type, structure, key directories.

## Components
One line per component: Name: path. Purpose. Key deps.

## Patterns
- Bullet per pattern observed

## Tests
Framework, pattern, run command.

## Docs Index
- [Name](docs/path) - Description
```

## Phase 4 - Risk scan

Run risk-detection skill patterns against codebase using Grep. Write findings to `risks.md`:

```markdown
# Risks
Next ID: RN | Last scan: DATE | Baseline: counts per level

## Critical
#R1 [CRITICAL] Description. file:line (found DATE)

## High
...

## Medium
...

## Low
...

## Resolved
```

## Phase 5 - Docs indexing

If `.vibe/docs/` contains files:
- Generate docs index section in `understanding.md`
- For heavy formats (PDF, images): generate companion `.md` with extracted text

## Phase 6 - Test framework selection

If test framework already detected from pre-compute: use it. Report which one.

If no framework detected: ask user with recommendation based on languages:
- Python -> "Recommend pytest (fast, fixtures, plugins). Or specify another."
- TypeScript/JavaScript -> "Recommend vitest (fast, ESM native). Or jest if preferred."
- Go -> `go test` (built-in, no choice needed)
- Rust -> `cargo test` (built-in, no choice needed)
- Multi-language -> ask per language

Record choice in `understanding.md` Tests section.

## Phase 7 - debug/ test harness

If `debug/` exists with user-written tests: ask "Merge existing tests into generated suite or start fresh?" Default: merge.

Create `debug/` structure based on languages:
- **Single language:** flat structure (`debug/conftest.py` or `debug/setup.ts` + test files)
- **Multi-language:** per-language subdirectories (`debug/python/`, `debug/typescript/`)

Generate initial tests covering:
- **Functionality:** happy path + error paths for key components identified in Phase 3
- **Edge cases:** empty inputs, boundary values, null handling for public APIs
- **Security:** check for patterns from risk-detection skill (hardcoded secrets, injection)

Run `debug/` suite to verify baseline passes.

## Phase 8 - Output summary

Report:
- Git: repo status (local/remote/none)
- Components: count identified
- Risks: count per impact level (CRITICAL/HIGH/MEDIUM/LOW)
- Patterns: count identified
- Tests: count generated, framework used
- Debug/: suite status (pass/fail, count)
