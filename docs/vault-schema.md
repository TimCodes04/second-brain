# Vault Schema — the data contract

Vault root: `<vault>` = `VAULT_PATH` in `config.env` (written by `/setup`; see *Configuration* in `CLAUDE.md`).

Every note the agent writes conforms to this document. Bases dashboards, `/study` planning, and `/connect` all depend on these fields being present and exact.

## Folder layout

```
<vault>/
├── Home.md                      # entry point; embeds Bases views
├── Inbox.md                     # quick-capture inbox; processed by /inbox (archive: log/inbox-archive.md)
├── Profile.md                   # current work/projects/interests; read by skills for relevance hooks
├── atlas/
│   ├── maps/                    # MOCs (maps of content), one per domain
│   └── bases/                   # .base dashboard files
├── palace/                      # mind palaces: one note per palace + _Palace Guide.md
├── projects/                    # computational mini-projects from /project (note + code + figures each)
├── courses/
│   └── <course_slug>/           # kebab-case slug, e.g. qm-sakurai
│       ├── _Course - <Title>.md # course home ("_" sorts it first)
│       ├── NN <Lesson Title>.md # lessons, zero-padded NN prefix
│       ├── cards/NN Cards.md    # one card file per lesson
│       ├── problems/NN Problems.md  # one problem set per lesson (see /problems)
│       ├── assets/              # generated figures (matplotlib SVGs)
│       └── quizzes/YYYY-MM-DD Quiz <scope>.md
├── concepts/                    # GLOBAL atomic concept notes (flat)
│   └── Inbox Cards.md           # the one exception: type: cards, deck #flashcards/inbox (courseless cards)
├── sources/                     # one note per source document/book/URL
├── log/
│   ├── study/                   # daily session logs: YYYY-MM-DD.md
│   ├── briefs/                  # daily briefs from /brief: YYYY-MM-DD.md
│   └── inbox-archive.md         # processed Inbox items with resolutions
└── templates/                   # note skeletons (mirrors repo templates/; excluded from all queries)
```

The repo's `scaffold/` folder holds the one-time files `/setup` copies into a new vault (Home, Inbox, Profile, Palace Guide, inbox archive, `atlas/bases/*.base`, `.gitignore`).

Principles:
- **Concepts are global, courses are local.** A concept note is created once, in `concepts/`, and linked from every course that touches it. Never create a course-local copy of a concept.
- **Cards never live inside lessons** — reading a lesson must not reveal answers.
- **Everything queryable is a frontmatter property.** Bases reads only properties and file metadata, never body text.

## Dates

`YYYY-MM-DD`, in the user's **local** timezone. "Today" always comes from the system clock (`date +%F`), never from assumption. Date arithmetic (e.g. "today + 5 days") uses the same local date.

## Frontmatter common to all types

```yaml
type: <see below>
summary: One sentence — what this note is about. Powers /connect. Always present.
created: YYYY-MM-DD
updated: YYYY-MM-DD        # bump on every modification
tags: []                   # optional; flashcard deck tags go in the BODY, not here
```

## Note types

### `type: course` — course home (`courses/<slug>/_Course - <Title>.md`)

```yaml
type: course
status: active             # planned | active | paused | done
depth: exam                # overview | exam | mastery
course_slug: qm-sakurai
lang: en
sources: ["[[Sakurai - Modern Quantum Mechanics]]"]
lessons_total: 14
lessons_done: 3            # count of lessons with status != unread — recompute, never increment blindly
started: 2026-01-10
target_date: 2026-07-15    # exam date or personal deadline; optional
```

Body: what/why paragraph → syllabus as a linked, ordered lesson list grouped by section (each entry: link + one-line objective summary) → quiz checkpoints noted between sections → `## Related`.

**Invariant:** any skill that changes a lesson's `status` recomputes its course's `lessons_done` (count of the course's lessons with `status != unread`) in the same run.

### `type: lesson` (`courses/<slug>/NN <Title>.md`)

```yaml
type: lesson
course: "[[_Course - Quantum Mechanics (Sakurai)]]"
course_slug: qm-sakurai
order: 2
status: unread             # unread | read | quizzed | mastered
mastery: 0                 # 0–100; written only by /quiz
last_quizzed:              # YYYY-MM-DD; empty until first quiz
concepts: ["[[Stern-Gerlach Experiment]]", "[[Spin-1/2 System]]"]
objectives: ["State what SG demonstrated", "Compute sequential SG outcomes"]
source_pages: "[[Sakurai - Modern Quantum Mechanics]] pp. 1–12"   # optional
```

Body structure is defined in `docs/pedagogy.md` (advance organizer → exposition → worked example → self-explanation prompts → retrieval checks → summary → Related).

**Status transitions:**
- `unread` → `read`: the user confirms reading (via /study or manually) or finishes a /teach session. Skills only ever set `read` on an `unread` lesson — never downgrade `quizzed`/`mastered` to `read`.
- → `quizzed`: after any /quiz covering the lesson (also directly from `unread`).
- ↔ `mastered`: re-evaluated after every quiz covering the lesson — `mastered` iff mastery ≥ 85 AND its two most recent **lesson scores** are both ≥ 80; otherwise `quizzed` (a mastered lesson that slips is demoted).

**Lesson score** (per quiz): the rubric-weighted percentage (0–100) of that lesson's questions within one quiz session — not the quiz's overall score. Every quiz log records it in its `## Result` table, which is where the "two most recent lesson scores" are read from.

**Mastery update rule (applied by /quiz):** if `last_quizzed` is empty (first quiz) → `mastery = lesson score`; otherwise `mastery = round(0.4 × old + 0.6 × lesson score)` — recent performance dominates.

### `type: concept` (`concepts/<Concept Name>.md`)

```yaml
type: concept
domains: [physics/qm]      # hierarchical domain tags, lowercase
courses: [qm-sakurai]      # slugs of courses referencing it; append, never remove
mastery:                   # 0–100; 10 × the grade (/10) of the most recent quiz question involving it; EMPTY until first quizzed
aliases: [Kommutator]      # incl. original-language source terms; used for dedup lookup
```

Body: 3–10 sentence explanation in your own words → key formula(s) if any → one concrete example → `## Related`. Atomic: ONE concept per note. Before creating, search `concepts/` filenames AND aliases; merge into the existing note if found.

### `type: cards` (`courses/<slug>/cards/NN Cards.md`, or `concepts/Inbox Cards.md`)

```yaml
type: cards
course_slug: qm-sakurai
lesson: "[[01 The Stern-Gerlach Experiment]]"
card_count: 12             # keep in sync when adding/removing cards
```

Body: first line after frontmatter is the deck tag, e.g. `#flashcards/qm-sakurai` (or `#flashcards/inbox` for courseless cards). Then the cards, blank-line separated, in Spaced Repetition plugin format:

- Inline: `Question::Answer` (single-line, non-math only)
- Multiline Q&A (required for anything containing LaTeX):
  ```
  Why does blocking SGz− then measuring SGx "destroy" the z-information?
  ?
  Measuring $S_x$ projects onto $\lvert S_x;\pm\rangle$, each a superposition
  of the $S_z$ eigenstates — see [[Sequential Measurements]].
  ```
- Cloze: `The SG experiment showed angular momentum is ==quantized==.`

The plugin appends scheduling state as `<!--SR:...-->` comments. **Never edit a line containing one** (see CLAUDE.md hard rule 1). A card without an SR comment is "new" (due immediately).

### `type: quiz` — quiz log (`courses/<slug>/quizzes/YYYY-MM-DD Quiz <scope>.md`)

```yaml
type: quiz
course_slug: qm-sakurai
date: 2026-03-05
lessons: ["[[01 The Stern-Gerlach Experiment]]", "[[02 Kets, Bras, and Operators]]"]
score: 68                  # 0–100 overall
weak_concepts: ["[[Sequential Measurements]]"]
format: mixed              # recall | problems | mixed
```

Body: per-question record — question, user's answer (verbatim or summarized), grade /10, feedback given, concept links. `## Result`: overall score plus a per-lesson table (lesson · lesson score · mastery old → new). Ends with `## Follow-up` listing cards spawned from errors.

- **One log per course.** A session that interleaves several courses writes one log into each course's `quizzes/` folder, each holding that course's questions and score.
- **Never overwrite a log.** If the filename exists (second quiz of the day on the same scope), append ` (2)`, ` (3)`, … to the name.

### `type: source` (`sources/<Source Name>.md`)

```yaml
type: source
source_kind: pdf           # pdf | book | web
path: <sources>/Sakurai - Modern Quantum Mechanics.pdf   # absolute file path, or the URL for web sources
pages_total: 570
ingested_ranges: ["ch1: 1-60"]   # append after each /ingest chunk — makes ingestion resumable
lang: en
```

Body: chapter map (from the /ingest survey pass) → notes on quality/edition → list of concept notes extracted from it.

### `type: moc` (`atlas/maps/<Domain> MOC.md`)

```yaml
type: moc
domain: physics
```

Body: curated, sectioned link lists — courses in this domain, key concepts grouped by subtopic, sources, `## Someday` (ideas filed by /inbox). MOCs are hand-curated structure; skills append new courses/major concepts under the right heading. A newly created MOC gets a link under `## Maps` in `Home.md`. (`Home.md` itself is `type: moc`, `domain: home`.)

### `type: problemset` (`courses/<slug>/problems/NN Problems.md`)

```yaml
type: problemset
course_slug: qm-sakurai
lesson: "[[01 The Stern-Gerlach Experiment]]"
problems_total: 5
next_retry:                # earliest pending retry date (YYYY-MM-DD), empty if none
```

Body: one `## Pn · <stars> <title>` section per problem — statement in a `> [!question]` callout, complete solution + grading rubric in a folded `> [!success]-` callout, then a machine-parsed status line, exactly one of:
`**Status:** unsolved` · `**Status:** failed (retry after YYYY-MM-DD)` · `**Status:** solved YYYY-MM-DD`.
`/study` and `/brief` find due re-attempts by grepping `retry after` and comparing dates. Titles describe the situation, never the technique needed (see pedagogy.md § Problem practice).

### `type: brief` (`log/briefs/YYYY-MM-DD.md`)

Common fields plus `date:`. Read-only digest produced by `/brief`: courses at a glance, today's priorities, inbox status, world-event connections. Re-running `/brief` on the same day replaces that day's brief. Not shown in Bases dashboards.

### `type: profile` (`Profile.md`, singleton)

Common fields only. Sections: `## Current work & projects`, `## Interests`, `## Background & strengths`. The user edits it directly or tells the agent to; any skill may update it on request. Skills read it for personal-relevance hooks (guardrail in pedagogy.md).

### `type: palace` (`palace/<Palace Name>.md`)

Common fields only. Body: route line, then one `### <Room>` per room in fixed walking order; one bullet per spot, exactly:
`- **<spot>** — "<image, user's words verbatim>" → encodes [[Target]] (<what precisely>) — added YYYY-MM-DD`
Target notes carry a back-reference in `## Related`. Imagery is user-authored only — see pedagogy.md § Mind palace.

### `type: project` (`projects/<name>/<Name>.md`)

```yaml
type: project
course_slug: qm-sakurai
lessons: ["[[07 The Quantum Harmonic Oscillator]]"]
status: active            # active | done | abandoned
```

Body: Goal / Cements / Success criteria → Plan (who writes what) → Results (embedded figures) → "What this taught me" (user's bullets) → Related. Code files live alongside the note.

### `type: reference` — long-form study/reference documents

For substantial standalone documents that aren't atomic concepts and belong to no course (e.g. `Research Group Overview.md`). Common fields only (`summary`, `domains`, `aliases` welcome). Not shown in Bases dashboards by design, but `/connect` includes them in its corpus. The system notes `Inbox.md`, `log/inbox-archive.md` and `palace/_Palace Guide.md` also use `type: reference`; `/connect` skips those three.

### `type: study-log` (`log/study/YYYY-MM-DD.md`)

```yaml
type: study-log
date: 2026-03-03
minutes: 45
courses: [qm-sakurai]
```

Body: what was planned vs. done (cards reviewed per deck, quiz results, lessons read), observations, and tomorrow's carry-over. A second session on the same day appends a `## Session 2` block (planned/done/notes/carry-over) and adds to `minutes` — never overwrite an existing log.

## Wikilink conventions

- Link a concept on its **first mention** in any note's prose; don't re-link every mention.
- Every lesson links its course home and its prev/next lesson (in the advance organizer / footer).
- Every note body ends with a `## Related` section: `- [[Other Note]] — one-line reason`. `/connect` maintains these; humans and other skills may add entries too.
- Source citations in prose: `([[Sakurai - Modern Quantum Mechanics]] p. 12)`.
- **Link targets must be unambiguous.** Obsidian resolves `[[Name]]` by filename alone, so two files with the same basename make links silently point at the wrong one. Before creating a lesson, source, or project note, check the whole vault for the basename and pick a more specific title if it is taken (`01 Introduction to Quantum Mechanics`, not `01 Introduction`). Card and problem files are never linked by bare name — every course has an `01 Cards.md` — so use the path form: `[[courses/<slug>/cards/01 Cards|01 Cards]]`.

## Reading FSRS state (for /study, /brief, /quiz)

Due-card state is read-only. **Card files** = every `.md` file outside `templates/` whose frontmatter has `type: cards` (i.e. `courses/*/cards/*.md` plus `concepts/Inbox Cards.md`). Grep those for `<!--SR:`. A card with no SR comment is new (= due immediately).

**Verified format** (extracted from the Spaced Repetition plugin source, v1.15.4, algorithm = FSRS, scheduling data stored in notes):

```
<!--SR:!fsrs,<due>,<interval>,<stability>,<difficulty>,<state>,<reps>,<lapses>,<learningSteps>,<lastReview>-->
```

- `<due>` and `<lastReview>` are full ISO-8601 UTC timestamps (e.g. `2026-09-10T14:23:05.000Z`), or `-` when unset.
- Sibling cards (e.g. multiple clozes in one line) concatenate segments inside ONE comment, separated by `!`: `<!--SR:!fsrs,…!fsrs,…-->`.
- Legacy SM-2 segments look like `!2026-09-10,4,270` (due date, interval, ease) — the plugin parses both.

**Parsing recipe:** match `<!--SR:(.+?)-->`, split the capture on `!` (drop empty pieces), split each segment on `,`.
- `fsrs` segment → field 1 (0-indexed) is the due timestamp (`-` = new).
- Any other segment (legacy SM-2) → field 0 is the due date `YYYY-MM-DD`; compare it to today's local date.
- **Cutoff:** a card is due today if its due timestamp ≤ the end of the user's *local* day expressed in UTC — e.g. `date -u -r $(date -j -v23H -v59M -v59S +%s) +%Y-%m-%dT%H:%M:%SZ` on macOS, `date -u -d "$(date +%F) 23:59:59" +%Y-%m-%dT%H:%M:%SZ` on Linux. Same-format ISO UTC strings compare lexicographically. (Using today's *UTC* date instead is wrong by up to a day away from UTC.)

Count per deck by the card file's deck tag.
