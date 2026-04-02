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
