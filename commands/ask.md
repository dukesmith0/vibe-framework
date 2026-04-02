---
description: Answer questions about this codebase using .vibe/ context and code search
argument-hint: <question>
---

1. Read `.vibe/` context via vibe-context skill (role: general). If no `.vibe/`: "No vibe context found. Run /vibe:init to analyze this codebase."
2. Check docs index in understanding.md. If question relates to an indexed doc, load it from `.vibe/docs/`.
3. Search actual codebase with Grep/Read for specific details.
4. Answer with file:line citations and code snippets where helpful.

Keep answers concise. Reference specific files and line numbers. If the answer reveals a pattern or decision worth recording, suggest: "Record this with `/vibe:note decision <summary>`."
