---
description: Import file or folder to .vibe/docs/ for reference - converts heavy formats, indexes automatically
argument-hint: <path>
---

1. Check `.vibe/` exists. If not: "No vibe context found. Run /vibe:init first."

2. Validate path exists (file or folder). If not found, report error and suggest similar paths if possible.

3. Copy to `.vibe/docs/`, preserving folder structure for directories.

4. For each file:
   - Text files (`.md`, `.txt`, `.json`, `.yaml`, `.csv`, etc.) -> copy as-is
   - Heavy formats (PDF, images, `.docx`, `.pptx`, etc.) -> copy original + generate companion `.md` with extracted text content, same filename
   - Large files (>10 pages or >50KB text): ask user "This file is [size]. Import full text or summary only?" Default: under 10 pages -> full, over 10 -> summary. Recommend splitting very large docs into focused sections.

5. Update docs index in `understanding.md`:
   - Add one-line entry per file: `- [Filename](docs/path) - One-line description`
   - Generate the description from the file's first paragraph or title

6. Report: files added, conversions made, index updated.
