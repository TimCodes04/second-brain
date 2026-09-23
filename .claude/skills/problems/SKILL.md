---
name: problems
description: Problem-solving practice — generate graded problem sets per lesson (worked → faded → independent), run LLM-graded attempt sessions in the terminal, and re-queue failed problems for spaced re-attempts.
argument-hint: "[course-slug] [--lessons 1,3] [--due] [--n 5]"
---

# /problems [<course-slug>] [--lessons 1,3] [--due] [--n <count>]

Problem-solving is a distinct skill from recall and is trained separately (see `docs/pedagogy.md` § Problem practice). Read that and `docs/vault-schema.md` (problemset schema) first. Vault: `<vault>`.

## Modes

- **`--due`** (default when due retries exist): collect all problems whose status line says `retry after <date>` with date ≤ today — across all courses if no slug given — and run a re-attempt session, interleaved.
- **New set** (default otherwise): generate a fresh problem set for the given lessons (default: the most recently read lessons without one).

## Problem set files

`courses/<slug>/problems/NN Problems.md` (NN = lesson number), created from `templates/problemset.md`, frontmatter per vault-schema (`type: problemset`, `course_slug`, `lesson`, `problems_total`, `next_retry`). Body per problem:

```markdown
## P1 · ★★ <short title — the situation, never the technique it needs>
> [!question] <full statement, LaTeX as needed>
> > [!success]- Solution
> > <complete worked solution + grading rubric (what earns which points)>

**Status:** unsolved
```

Status line values (machine-parsed — keep the exact format): `unsolved` · `failed (retry after YYYY-MM-DD)` · `solved YYYY-MM-DD`.

## Generating a set

3–6 problems per lesson, difficulty ★ to ★★★, following the guidance-fading arc: first a worked-example completion (★), middle problems with scaffolding removed (★★), final ones independent — including **one transfer problem** applying the idea in an unfamiliar context (★★★). Every problem grounded in the lesson's concepts (link them); solutions complete enough to learn from alone; never fabricate numbers a real computation wouldn't give (compute them, with matplotlib/Bash if needed).

## Attempt session (terminal)

1. One problem at a time: show the statement, NOT the solution. The user answers with a final result and/or solution sketch; they may also say "hint" (give the smallest useful hint, note it) or "skip".
2. Grade /10 against the rubric, partial credit, elaborative feedback (where the approach diverged, why the correct step works, link concepts).
3. Update the status line: score ≥ 7 → `solved YYYY-MM-DD`; score < 7 → `failed (retry after <today + 3–7 days>)` (harder problems get the shorter delay; a second consecutive fail — the status line already said `failed` before this attempt — gets +3 days and a pointer to re-read the lesson section first). Recompute `next_retry` in frontmatter as the earliest remaining `retry after` date in the file (empty if none).
4. A problem that needed a hint but scored ≥ 7 → `failed (retry after <today+7>)` once more — solved-with-hint is not yet solved.
5. Close out: session summary (scores, what's re-queued for when), append one line to today's study log if it exists, and tell `/study` users what changed.

Invariants: never reveal a solution before an attempt (or explicit "show me"); never modify `<!--SR:` lines anywhere; bump `updated:` on edited files.
