---
description: Generates multi-line flashcards from conversation or notes and saves them along with the document.
category: thinking
triggers_en: ["generate flashcards", "create flashcards", "make flashcards", "create flashcard"]
---

Use the obsidian-second-brain skill. Execute `/obsidian-flashcard $ARGUMENTS`:

Generates flashcards from recent context or a specified note and saves them to your spaced repetition decks.

1. Read `_CLAUDE.md` first if it exists in the vault root.
2. Determine the topic or note from the argument, or infer it from the most recent conversation context. If a specific note is given, read its contents.
3. Extract the core concepts and generate 5-10 high-quality flashcards using the multi-line Spaced Repetition plugin syntax: `Question\n?\nAnswer`.
6. Ensure the note contains the `#flashcards` or `#flascards/$subdeckname` tag so the plugin detects it.
7. Log the creation in `Logs/YYYY-MM-DD.md` and link the new flashcard note in today's daily note.

Reviewing your notes via flashcards helps solidify long-term memory.

---

**AI-first rule:** Every note created or updated by this command MUST follow `references/ai-first-rules.md` — `## For future Claude` preamble, rich frontmatter (`type`, `date`, `tags`, `ai-first: true`, plus type-specific fields), recency markers per external claim, mandatory `[[wikilinks]]` for every person/project/concept referenced, sources preserved verbatim with URLs inline, and confidence levels where applicable. The vault is for future-Claude retrieval — not human reading.
