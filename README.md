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
├── current.md         # Active task: plan, progress, and state
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
