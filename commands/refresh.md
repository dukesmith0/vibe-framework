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

# Test framework
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(test -f vitest.config.ts -o -f vitest.config.js && echo "test:vitest") || \
(test -f jest.config.ts -o -f jest.config.js && echo "test:jest") || \
(node -e "const p=require('./package.json');if(p.scripts?.test){console.log('test:'+p.scripts.test);process.exit(0)}else{process.exit(1)}" 2>/dev/null) || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"

# Playwright
(test -f playwright.config.ts -o -f playwright.config.js) && echo "playwright:yes" || \
(grep -q playwright package.json 2>/dev/null && echo "playwright:yes") || echo "playwright:no"
```

If no `.vibe/`: "No vibe context found. Run /vibe:init first."

## Phase 1 - Reconcile understanding.md

- Check all components listed against filesystem. Remove entries for deleted files.
- Scan for new entry points (main.*, index.*, server.*, app.*, routes/*, handlers/*). Add to understanding.md.
- Check package manifests (package.json, pyproject.toml, go.mod, Cargo.toml) against Stack section. Update if deps changed.
- Check config files (.env.example, docker-compose, config/) for changes. Update if needed.
- If >30% of listed files changed: do full re-scan (like init Phase 3). Otherwise delta-update only changed sections.

## Phase 2 - Reconcile debug/

- Validate all test imports. Remove tests that reference deleted source files.
- Check if test framework changed (different from what debug/ uses). If so: regenerate debug/ with new framework. Report the switch.
- Scan for untested new code. Generate missing tests.

## Phase 3 - Docs check

Scan `.vibe/docs/` for new/changed files. Update docs index in understanding.md. Generate `.md` companions for new heavy formats.

## Phase 4 - Risk re-scan

Run risk-detection skill patterns. Update `risks.md`:
- New risks: add with next sequential ID and impact level
- Resolved risks (file:line no longer matches): move to Resolved section with date
- Report delta per impact level

## Phase 5 - Run debug/ suite

Run full debug/ test suite to verify current state. Report results.

## Phase 6 - Report

Summarize: what changed in understanding, files removed, tests added/removed, new risks found, risk delta, test results.
