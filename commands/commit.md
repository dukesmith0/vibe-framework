---
description: Smart commit with test gate, .vibe/ sync, and optional PR creation
argument-hint: [message hint]
---

## Pre-compute

```bash
# Commit state
git status -s 2>/dev/null
git diff --staged --stat 2>/dev/null
git diff --stat 2>/dev/null
git log --oneline -5 2>/dev/null
git branch --show-current 2>/dev/null
git remote -v 2>/dev/null | head -2

# Testing
test -f pytest.ini && echo "test:pytest" || \
(grep -q pytest pyproject.toml 2>/dev/null && echo "test:pytest") || \
(test -f vitest.config.ts -o -f vitest.config.js && echo "test:vitest") || \
(test -f jest.config.ts -o -f jest.config.js && echo "test:jest") || \
(node -e "try{const p=require('./package.json');console.log(p.scripts?.test?'test:'+p.scripts.test:'test:none')}catch{}" 2>/dev/null) || \
(test -f go.mod && echo "test:go") || \
(test -f Cargo.toml && echo "test:cargo") || \
echo "test:none"
```

If no changes staged or unstaged: "No changes to commit." Exit.
If no git: "No git repository. Run `/vibe:init` to set up."

## Phase 1 - Test gate

Run `debug/` suite (if exists). If failures: report and do NOT proceed. Tests must pass.
Run project tests (if configured). Same gate. Both must pass before committing.

## Phase 2 - Analyze

What changed and why. Group into logical commit(s). Read `plans.md` and `current.md` for context on what was being worked on.

## Phase 3 - .vibe/ sync (read AND write)

Check and WRITE updates to these files:
- `understanding.md`: if diff affects architecture/patterns (new top-level dirs, entry points, frameworks), update the relevant sections now.
- `bugs.md`: if diff resolves any open bugs (code at file:line fixes the described issue), move entry to Resolved section with date and regression test ref. Write the change now.
- `risks.md`: if diff resolves any risks (file:line pattern no longer matches), move entry to Resolved section with date. Write the change now.
- `debug/`: if a bug was resolved but has no regression test in debug/, generate one now.

## Phase 4 - Draft message

- Concise. Follow project commit style (look at `git log --oneline` for convention).
- First line under 72 chars. Add body if "why" isn't obvious.
- Use conventional commits (feat:/fix:/chore:) only if project already uses them.
- **Do NOT add Co-Authored-By lines. User's attribution setting handles this.**

## Phase 5 - Approve

**[STOP]** Present staged changes + draft message for approval. User can edit message.

## Phase 6 - Execute

- Stage relevant files if not already staged. Commit with approved message.
- Push to remote if remote exists.
- If push fails: check if remote is ahead (`git fetch && git status`). Pull if needed, retry.
- If pre-commit hook fails: display error. Ask user to fix or suggest `/vibe:review` to auto-fix style issues. Re-stage, create NEW commit (never amend).

## Phase 7 - PR offer

If on feature branch (not main/master) and remote exists:
"Create a pull request? [yes/no]"
If yes: `gh pr create` with title from commit message and body summarizing changes.
