---
name: card
description: Add, edit, or regenerate flashcards for a lesson or concept note, following the card-writing rules and preserving FSRS scheduling state on untouched cards.
argument-hint: "<note-or-topic> [--add \"what\"] [--edit] [--regenerate]"
---

# /card <note-or-topic> [--add "<what>"] [--edit] [--regenerate]

Card surgery on one card file. Read `docs/pedagogy.md` (card rules) and `docs/vault-schema.md` (cards schema) first. Vault: `<vault>`.

1. **Locate the card file**: for a lesson, `courses/<slug>/cards/NN Cards.md`; for a concept or free topic, the inbox file `concepts/Inbox Cards.md` (deck `#flashcards/inbox`; create from `templates/cards.md` if missing). If ambiguous, list candidates and ask.
2. **Mode:**
   - `--add`: append new cards for the requested fact/idea (default 1–3, atomic, per card rules). Typical after a `/tutor` insight.
   - `--edit`: show the current cards numbered; user says what to change. Rewrite only the card(s) named.
   - `--regenerate`: propose a fresh card set from the source note; show a diff-style summary (kept / replaced / new) and **confirm before writing** — replaced cards lose their FSRS history by design.
3. **Invariants:** never modify a line containing `<!--SR:` except when the user explicitly regenerates/edits that exact card (removal of the comment = deliberate reset — say so). Keep the deck tag as the first body line. Update `card_count` and `updated:`.
4. Report what changed and how many cards are now due-new.
