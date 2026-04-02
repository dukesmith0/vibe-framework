# vibe-framework
Last: 2026-04-01 | 26 files | Markdown, JSON, Bash

## Stack
Claude Code plugin system (markdown frontmatter + prompt body). Bash for pre-compute. Git/gh CLI for version control.

## Architecture
Claude Code marketplace plugin. Single plugin, flat structure. Commands in commands/, agents in agents/, skills in skills/*, hooks in hooks/. Config in .claude-plugin/ and .claude/.

## Components
Plugin manifest: .claude-plugin/plugin.json. v3.0.0, author dukesmith0, MIT license.
Marketplace: .claude-plugin/marketplace.json. Registry entry for vibe-marketplace.
Settings: .claude/settings.local.json. WebSearch permission.

Commands (13): init, plan, go, tdd, oneshot, ralph, review, commit, ask, note, add, refresh, help.
- Workflow commands (go, oneshot, ralph, tdd): dispatch subagents, multi-phase, auto-update .vibe/. Plan and progress tracked in current.md.
- Utility commands (ask, note, add, help, refresh): lightweight, read/append operations.
- Lifecycle commands (init, commit): create .vibe/, gate commits on tests.
- Planning (plan): EnterPlanMode, clarifying questions, execute-or-defer. Plans written to current.md or deferred to future.md.

Agents (4): explorer (investigate), engineer (implement), reviewer (adversarial review), tester (run tests).
- All use vibe-context skill for context loading. All have subagent_type frontmatter.
- Engineer: 4-state output (DONE/DONE_WITH_CONCERNS/BLOCKED/NEEDS_CONTEXT), 3-fix rule.
- Reviewer: two-stage (spec compliance then quality), CRITICAL/HIGH/MEDIUM/LOW output.
- Tester: project tests + debug/ suite + linters. No LSP tool.

Skills (2): vibe-context (auto-load .vibe/ per role, API lookup chain), risk-detection (pattern scan, write to risks.md).
Hooks (1): Stop (session-end warning for active tasks and uncommitted .vibe/ changes).

## Patterns
- Pre-compute bash blocks in workflow commands (git state, test framework, playwright, dev server)
- Subagent dispatch with fallback: `subagent_type="vibe:X"` or `"general-purpose"`
- Reviewer + tester dispatched in parallel (single message) in go, oneshot, ralph, review
- [STOP] markers for user approval gates, [STOP IF CRITICAL/HIGH] for conditional gates
- .vibe/ auto-updates: every workflow command reads and writes relevant .vibe/ files
- Bug IDs (#N) and risk IDs (#RN) with CRITICAL/HIGH/MEDIUM/LOW impact levels
- decisions.md entries tagged [USER] or [CLAUDE] with assumptions section
- TodoWrite for real-time phase progress in all workflow commands
- Plan mode (EnterPlanMode) for brainstorming in /vibe:plan

## Tests
No test framework (this is a plugin of markdown prompt files, not executable code).
debug/ harness: not applicable for this project (no source code to test).

## Docs Index
- [Redesign Spec](docs/2026-04-01-vibe-framework-redesign.md) - Full v3.0 design spec with .vibe/ structure, commands, agents, skills, token efficiency strategy
- [Implementation Plan](docs/2026-04-01-vibe-framework-rewrite.md) - 16-task implementation plan with full file content for ground-up rewrite
