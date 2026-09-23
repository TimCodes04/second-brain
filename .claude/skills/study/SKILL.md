---
name: study
description: Daily study driver — reads due-card state and course progress, plans today's session (reviews → quiz → new material), runs it with the user, and writes the study log.
argument-hint: "[--minutes 45]"
---

# /study [--minutes <N>]

The daily loop. Read `docs/pedagogy.md` (session rules) and `docs/vault-schema.md` first. Vault: `<vault>`.

## 1. Gather state (read-only)

- **Due cards:** in every card file (definition, parsing recipe, and the local-day cutoff are at the bottom of `docs/vault-schema.md`), grep `<!--SR:` and count per deck: due today/overdue, plus cards with no SR comment (= new).
- **Due problems:** grep `retry after` in `courses/*/problems/*.md`; collect problems with retry date ≤ today.
- **Inbox:** count pending items in `Inbox.md`.
- **Courses:** active course homes (`lessons_done`/`lessons_total`, `target_date`); lessons `status`/`mastery`/`last_quizzed`; quiz logs from the last 14 days; yesterday's study log carry-over.

## 2. Plan the session

Budget: `--minutes` or ~45 by default. Order per pedagogy: ⓪ process pending Inbox items (follow the `/inbox` procedure) ① due-card review in Obsidian (~1 min per 4 cards) ② retrieval work in the terminal — due problem re-attempts (follow the `/problems --due` procedure) and/or `/quiz --weak` on the weakest/stalest lessons ③ new material (next unread lesson, `/learn --continue`, or next `/ingest` chunk). Near a `target_date` (< 3 weeks), shift ③ toward cumulative interleaved quizzing instead of new content. Print the plan, e.g.:

> ① Obsidian: review 23 due cards (`qm-sakurai` 17, `lagrangian` 6) — do this now, then come back.
> ② `/quiz qm-sakurai --weak` (L03 mastery 45, stale since Aug 12) — I'll run it here.
> ③ Read [[05 Perturbation Theory]] (~25 min), then mark it read.

## 3. Run it

Wait for the user to do the Obsidian review (ask for the plugin's end-of-review count as a sanity check). Run the quiz inline (follow the quiz skill's procedure). For reading items, confirm afterwards and set that lesson's `status: read` if it was `unread` (never downgrade a quizzed/mastered lesson).

## 4. Close out

Write `log/study/YYYY-MM-DD.md` from `templates/study-log.md` (planned vs. done, minutes, carry-over). If today's log already exists (second session), append a `## Session N` block and add to `minutes` — never overwrite. Recompute course `lessons_done` if statuses changed. End with one line on tomorrow's priority.
