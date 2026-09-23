---
name: teach
description: Teach a lesson as an adaptive Socratic dialogue instead of static reading — pretest first, build on what the user already knows, and record the session into the lesson note.
argument-hint: "<lesson or topic> [--course slug]"
---

# /teach <lesson or topic> [--course <slug>]

Dialogue is the primary encoding mode; reading is the fallback. Read `docs/pedagogy.md` and `docs/vault-schema.md` first. Vault: `<vault>`.

## 1. Prepare (silent)

Locate the lesson (full or stub) and read it, its concepts, cited source pages, and the course home. Read `Profile.md` for at most one honest relevance hook (guardrail in pedagogy.md — seasoning, never steering). Note the lesson's objectives; they define what the dialogue must cover — **dialogue changes the delivery, never the coverage**.

## 2. Pretest

3–5 rapid diagnostic questions spanning the objectives, one message, quick answers expected. Tell the user wrong answers are free — pretesting improves the learning that follows even when answers miss. Use the result to set pace: segments the user demonstrably knows get compressed to a confirmation, not a walkthrough.

## 3. Teach the dialogue

Segment by segment:
- Open each segment with a **prediction question** ("what should happen if…?"), let the user commit, then teach *from their answer* — build on what's right, name precisely what's wrong.
- Run the worked example as a **guided problem**: the user attempts each step, you steer with the smallest useful hint.
- Comprehension-check every few minutes with one-line retrieval questions; adapt live — struggling → smaller steps and a concrete analogy; cruising → compress and pose a transfer question instead.
- Keep it warm, precise, two-way; never lecture for more than a few paragraphs without handing the turn back.

## 4. Close the loop

The user summarizes the lesson in their own words; correct the summary. Then write the artifacts:
- **Stub lesson** → write the full lesson note now (standard pedagogy structure and full coverage — the note must stand alone for later re-reading), plus its card file if missing.
- **Existing lesson** → leave the exposition intact.
- Either way, append a `## From our session — YYYY-MM-DD` section to the lesson note: pretest outcome, 2–6 bullets of *this user's* wrong turns, aha-moments, and the analogies that landed (these personalized traces are gold for later review).
- If the lesson was `unread`, set `status: read` and recompute the course's `lessons_done`; a `quizzed`/`mastered` lesson keeps its status. Bump `updated:`. Mastery stays untouched — `/quiz` owns it; recommend quizzing in 1–3 days, not today (spacing beats immediate testing).
