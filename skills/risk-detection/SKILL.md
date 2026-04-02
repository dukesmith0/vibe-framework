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
