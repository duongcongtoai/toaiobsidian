---
description: Promote a WIP (Work In Progress) note into the permanent Knowledge base. Extracts finalized code, concepts, and answers, synthesizes them into long-term memory, and archives the draft.
category: thinking
triggers_en: ["graduate this wip", "ingest wip", "move wip to knowledge", "finalize draft"]
---

Use the obsidian-second-brain skill. Execute `/obsidian-wip-graduate $ARGUMENTS`:

The optional argument is the WIP title, tag, or keyword. If not provided, scan the `WIP/` folder for notes tagged `#wip` or with `status: ready-to-ingest` and present them for selection.

1. Read `_CLAUDE.md` first if it exists in the vault root.
2. Find the WIP to graduate:
   - If argument given: search `WIP/` for a matching file (fuzzy match).
   - If no argument: list recent WIP notes (last 14 days) and ask the user to pick one.
3. Read the full WIP note and any linked notes for context. Pay special attention to coding demos, answered questions, and resolved research.
4. Research the vault for related content:
   - Search `Knowledge/` for existing concept notes that overlap or should be updated.
   - Look for related `Projects/` or `Dev Logs/` where this research originated.
5. Synthesize and Graduate to Long-Term Memory:
   - **Spawn Subagents (or execute serially):**
     - For net-new concepts/tools/patterns: Create new notes in `Knowledge/` with full context, code snippets, and explanations derived from the WIP.
     - For existing concepts: REWRITE the existing notes in `Knowledge/` to integrate the new learnings, code demos, or clarified answers. Do not just append; synthesize.
   - Ensure all new/updated notes follow the AI-first rules (preamble, frontmatter, recency markers).
6. Update the original WIP note:
   - Change frontmatter `status` to `graduated`.
   - Add a `Graduated To:` section at the top of the file containing `[[wikilinks]]` to the newly created or updated `Knowledge/` notes.
   - Move the file to `WIP/_archived/` (create the folder if it doesn't exist) to keep the active WIP folder clean, preserving the raw history.
7. Update today's daily note (`Logs/` or root depending on structure):
   - Add an entry stating that the WIP was graduated and list the Knowledge notes that were impacted.
8. Report back to the user:
   - What WIP was processed.
   - **New Knowledge pages created** (list).
   - **Existing Knowledge pages rewritten** (list with what changed).

The WIP doesn't die - its raw timeline is preserved in the archive, but its verified conclusions are extracted so future-Claude can retrieve them reliably as ground-truth facts.

---

**AI-first rule:** Every note created or updated by this command MUST follow `references/ai-first-rules.md` - `## For future Claude` preamble, rich frontmatter (`type`, `date`, `tags`, `ai-first: true`, plus type-specific fields), recency markers per external claim, mandatory `[[wikilinks]]` for every person/project/concept referenced, sources preserved verbatim with URLs inline, and confidence levels where applicable. The vault is for future-Claude retrieval - not human reading.

**Anti-fabrication:** Search exhaustively before claiming any note, person, or file is absent - false absence is the most common failure mode - and never invent facts, entities, or dates (mark unknowns as `TBD`). See the anti-fabrication and search-completeness hard rules in `references/ai-first-rules.md`.