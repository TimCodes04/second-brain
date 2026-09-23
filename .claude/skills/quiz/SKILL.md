---
name: quiz
description: Run an interactive LLM-graded retrieval quiz in the terminal over one course or a set of lessons, then update mastery, write the quiz log, and spawn targeted flashcards from every error.
argument-hint: "[course-slug] [--lessons 1,3] [--weak] [--n 8]"
---

# /quiz [<course-slug>] [--lessons 1,3] [--weak] [--n <questions>]

Interactive retrieval practice — the core of the testing effect. Read `docs/pedagogy.md` (quiz rules) and `docs/vault-schema.md` (mastery update rule, quiz-log schema) first. Vault: `<vault>`.

## Scope selection

- `--lessons`: exactly those lessons of the course.
- `--weak`: lessons with `mastery < 60`, plus stale ones (`last_quizzed` > 21 days, `mastery < 85`).
- Default: interleaved mix across the course's `status: read|quizzed|mastered` lessons, weighted toward low mastery and staleness. No course given → mix across all active courses (max interleaving).
- Read the scoped lessons AND their concept notes to ground questions. Default length: 6–10 questions (`--n` overrides).

## Session loop (one question at a time)

1. Before asking, privately write a 10-point rubric for the question.
2. Ask conversationally. Mix formats per pedagogy.md: free recall, explain-why, worked problems (user answers in text/LaTeX-ish), transfer questions. Never multiple choice.
3. Every 2–3 questions, ask for a confidence rating (1–5) before grading — flag high-confidence misses.
4. Grade /10 against the rubric with partial credit. Give elaborative feedback: correct answer, why, misconception note if relevant, link to the concept note. If the missed knowledge is encoded in the user's mind palace (grep `palace/` for a link to the concept), add the location as a retrieval cue ("this lives in your palace — <room>, <spot>") — cue the place, never recite their image. Then next question.
5. The user can stop anytime ("stop", or the equivalent in their language) — finish the session bookkeeping with what was answered.

## After the last question

All rules below are defined in `docs/vault-schema.md` (lesson type) — apply them exactly.

1. Overall score /100 (weighted by rubric points), plus a **lesson score** (0–100) for each covered lesson from its own questions only.
2. Write the quiz log per `templates/quiz-log.md` at `courses/<slug>/quizzes/YYYY-MM-DD Quiz <scope>.md` — per-question record, `weak_concepts` (anything graded ≤ 5, plus high-confidence misses), and the per-lesson `## Result` table. A session spanning several courses writes **one log per course**. If the filename already exists, append ` (2)`, ` (3)`, … — never overwrite a log.
3. Update each covered lesson's frontmatter, reading `last_quizzed` *before* overwriting it: `mastery` (first quiz, i.e. `last_quizzed` empty → lesson score; else `round(0.4×old + 0.6×lesson score)`), then `last_quizzed` = today, then `status` (`mastered` iff mastery ≥ 85 and the last two lesson scores — this one and the previous quiz log's — are ≥ 80, else `quizzed`; this also demotes a slipped `mastered`). Recompute each affected course's `lessons_done`. Set each involved concept note's `mastery` to 10 × its most recent question grade.
4. **For every failed item (≤ 5/10): append 1–2 new targeted cards** to the relevant lesson's card file (or `concepts/Inbox Cards.md`, deck `#flashcards/inbox`, if courseless), attacking exactly the gap. New cards have no SR comment → due immediately. Never touch existing card lines with `<!--SR:` comments; bump `card_count`.
5. Report: score, per-lesson mastery changes, cards spawned, and what `/study` should prioritize tomorrow.
