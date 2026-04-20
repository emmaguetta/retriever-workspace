---
description: Run git status across all 4 workspace repos and flag issues
---

Run `git status --short` and `git log --oneline @{u}..HEAD 2>/dev/null` in each of these repos:
- `GTMbuilder/`
- `retriever-apps/`
- `retriever-marketing-website/`
- `Retriever-WEB-EXTRACTION/`

Also run the same in the workspace root (`.`) to report on workspace-level changes.

Summarize in a table : repo | branche | modifs locales | commits non pushés. Flag en rouge tout repo avec modifs non commitées ou commits non pushés.
