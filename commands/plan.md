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

## Phase 1 - Enter plan mode

- Use EnterPlanMode to prevent code writes during brainstorming.
- Read `understanding.md` for project context.
- Read `future.md` to check if goal matches an existing future plan. If match found, surface it: "Found a related plan in future.md: [summary]. Build on this?"
- Read `decisions.md` for relevant prior decisions.

## Phase 2 - Clarifying questions

Ask questions one at a time. Prefer multiple choice when possible.
Focus on: purpose, constraints, success criteria, scope, integration points.
Keep going until you understand the goal well enough to propose approaches.
Do NOT ask more than 5 questions unless the goal is genuinely ambiguous.

Example:
> "What's the primary goal of this feature?"
> a) Performance improvement
> b) New user-facing functionality
> c) Internal tooling/developer experience
> d) Bug fix or reliability improvement

## Phase 3 - Propose approaches

Present 2-3 approaches with trade-offs. Lead with your recommendation and explain why.

Format per approach:
```
**Approach A: [Name]** (recommended)
- Description: [2-3 sentences]
- Pros: [bullet list]
- Cons: [bullet list]
- Complexity: [Low/Medium/High]
```

## Phase 4 - Present plan

After user chooses approach, present the plan:

```markdown
## Goal
[One sentence]

## Approach
[2-3 sentences describing chosen approach]

## Success Criteria
- [ ] [Specific, measurable outcome]
- [ ] [Specific, measurable outcome]

## Tasks
- [ ] [Actionable step]
- [ ] [Actionable step]
```

Record user's chosen approach as `[USER]` decision in `decisions.md`:
`DATE [USER] Chose [approach name] for [goal]. Why: [user's reasoning or "recommended by Claude"].`

## Phase 5 - Execute or defer

- ExitPlanMode.
- [STOP] Ask: "Execute now or save for later?"

**Execute now:**
- Check `current.md`. If active task: "Active task found: [description]. Resume, abandon, or start new?"
  - Resume: return to active task
  - Abandon: archive to `decisions.md` Plan Archive with outcome "abandoned", clear `current.md`, clear `plans.md`
  - Start new: archive current task and plan, clear both, proceed
- Archive existing plan in `plans.md` if any (to `decisions.md` Plan Archive). Then clear `plans.md`.
- Write new plan to `plans.md` (with Goal, Approach, Success Criteria, Tasks sections).
- Update `current.md` with initial progress checklist (same format as go Phase 5, starting at phase 6).
- Automatically begin `/vibe:go` workflow at Phase 4 (implement). Plan already written by this command, so go skips Phases 1-3 (entry routing, explore, plan+approve). Go reads `plans.md` for the plan and `current.md` for state.

**Save for later:**
- Append to `future.md`: `- [Goal summary]. [Approach chosen]. [Priority/context].`
- Return to user prompt.
