# Lessons Learned

## One skill per library, not per API section

**Date:** 2026-02-28
**Context:** Creating Sharp image processing skills
**Mistake:** Split Sharp into 7 separate skill directories (one per API section: constructor, metadata, output, resize, composite, effects, colour). User corrected: it should be one skill in one directory.
**Root cause:** Over-engineering the modularity. Sharp is a single coherent library — splitting it into many skills fragments the knowledge and makes it harder to use.
**Rule:** When creating a skill for a library/tool, keep it as **one skill folder** with one `SKILL.md`. Use `references/` subdirectory for detailed API sections that would make the main file too long. Only split into multiple skills if the domains are truly independent (e.g., different tools, different APIs, different use cases that would never overlap).
**Pattern:** `skill-name/SKILL.md` (core guidance, <500 lines) + `skill-name/references/*.md` (detailed API docs loaded on demand).
