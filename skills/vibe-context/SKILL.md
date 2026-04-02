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

## Test Coverage Reminder

When an engineer or implementation phase completes (not per-file, once per phase):
- Check if changed source files have corresponding tests in `debug/`
- If any changed file lacks a test: flag it before reporting phase complete
- This check runs at phase boundaries, not after every file write

## Token Efficiency Rules

When writing to .vibe/ files, follow these rules:
- One entry, max two lines. No prose paragraphs.
- Abbreviate paths when unambiguous.
- Use IDs for cross-reference (#3, #R2) instead of restating.
- Dates as YYYY-MM-DD. Strip filler words.
- Archive resolved entries when >20 accumulate.
