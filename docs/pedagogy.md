# Pedagogy — the learning-science rules

Single source of truth for how this system teaches. Every skill that generates lessons, cards, quizzes, tutoring, or schedules follows these rules. They are grounded in the cognitive-science literature: **retrieval practice and spaced practice are the two highest-utility techniques**; interleaving, elaborative interrogation, and self-explanation are moderate-utility; rereading and highlighting are low-utility and are never what this system produces.

## Principles (apply everywhere)

1. **Retrieval before re-exposure.** Any session that touches known material starts by making the user retrieve (quiz, free recall, "explain it back") before showing them anything.
2. **Spacing via FSRS.** Card scheduling belongs to the Spaced Repetition plugin's FSRS algorithm. The agent never invents its own card schedule; it reads due-state and plans sessions around it.
3. **Desirable difficulty.** Prompts must require *generation* of an answer, not recognition. If a card can be answered by "yes/no" or by eliminating options, rewrite it.
4. **Errors are fuel.** Every failed quiz item or card lapse produces a targeted follow-up: 1–2 new cards attacking exactly the gap, plus elaborative feedback explaining *why* the right answer is right.
5. **Elaboration and connection.** New knowledge is always tied to something already known: prerequisites linked in lessons, analogies made explicit, `## Related` sections maintained.
6. **Interleaving.** Quizzes and study sessions mix topics/lessons rather than blocking on one, weighted toward weak and stale material.
7. **Calibration.** In quizzes, occasionally ask for a confidence rating before revealing the grade — miscalibrated confidence (high confidence + wrong) is flagged in feedback and weighted extra in `weak_concepts`.
8. **Pretest before teaching.** A short diagnostic before generating or teaching material serves twice: failed retrieval attempts prime the encoding that follows (pretesting effect), and demonstrated knowledge gets compressed instead of re-taught — no lesson re-teaches what the user just proved they know.
9. **Dialogue over monologue.** One-on-one adaptive tutoring is the strongest known intervention; when the user is present, prefer teaching as dialogue (/teach, /tutor, /teachback) over handing them text. Written lessons remain complete — they are the durable record, not the primary delivery.

## Personal relevance (used by /learn, /teach, /brief, /project, /connect)

Self-relevant material is remembered better, so skills read `Profile.md` (current work, projects, interests) and add honest hooks — one per lesson at most, a real conceptual connection stated plainly. **Guardrail:** relevance is seasoning, never steering. Lessons stay factually correct, complete, and true to the field's canon; content is never cut, reordered, or biased toward the user's interests. A lesson with zero genuine hooks gets zero hooks.

## Mind palace (used by /palace, referenced by /quiz)

The method of loci works through *self-generated* vivid imagery anchored to a well-known spatial route. Therefore: the user invents every image and phrases it themselves; the system records verbatim, links mnemonics to knowledge notes, runs rehearsal walks (spaced like any retrieval practice), and cues *locations* — never recites the user's imagery for them during retrieval. Palace-worthy content: ordered lists, enumerations, constants, classification families. Understanding-based knowledge belongs in lessons and cards, not the palace.

## Card rules (used by /learn, /ingest, /card, /quiz)

- **Atomic:** one fact, relation, or step per card. A derivation becomes several cards (key step, why the step is valid, final result), never one monster card.
- **Format choice:**
  - Cloze (`==...==`) for definitions, statements, and named facts.
  - Multiline Q&A (`?` separator) for conceptual questions, "explain why", and **anything containing LaTeX** (inline `::` cards break too easily around math).
  - For formulas, prompt for generation: "State the canonical commutation relation for position and momentum" → `$[\hat x, \hat p] = i\hbar$`. Don't cloze fragments inside `$...$`.
- **No yes/no questions. No multiple choice on cards.**
- **Both directions when meaningful** (term→definition and definition→term) but only if the reverse is a fact worth knowing, not filler.
- **Answer side links the concept note** (`see [[Concept]]`) so a lapse leads to re-study material in one click.
- **5–15 cards per lesson.** More than that means the lesson covers too much or the cards aren't atomic.
- Card wording must be **context-free**: understandable months later without the lesson open ("In the Stern-Gerlach experiment, …", not "In this experiment, …").

## Lesson structure (used by /learn, /ingest)

Every lesson follows this arc:

1. **Advance organizer** (`> [!abstract]` callout): what this lesson covers, why it matters, where it fits — with links to prerequisite concepts and prev/next lessons.
2. **Exposition** with concepts linked on first mention. Precise but conversational; equations in LaTeX; every new symbol named in words. Build from a concrete phenomenon or question, not from the formalism.
3. **≥ 1 fully worked example** — presented as a problem in a `> [!question]` callout with the full solution in a **folded** `> [!success]-` callout ("try before unfolding").
4. **Self-explanation prompts** woven into the exposition: "Before reading on, predict what happens if …".
5. **Check yourself:** 3–5 retrieval questions at the end, answers in one folded callout. These are *session* retrieval; the durable versions live in the lesson's card file.
6. **Summary + `## Related`.**

Lesson length: one sitting (~20–40 min of reading). Split rather than bloat.

## Quiz rules (used by /quiz)

- One question at a time, conversational, in the terminal. Mix of: free recall ("state and explain…"), explanation ("why does…"), worked problems (user answers in text/LaTeX-ish), and transfer ("apply X to this new setup…").
- **Never multiple choice for physics or derivations.**
- Grade each answer /10 against a rubric you write before asking; partial credit with explicit reasoning. Feedback is elaborative: correct answer, *why*, common-misconception note if relevant, link to the concept note.
- Session composition: interleave across the scope's lessons; weight items toward `mastery < 60` and `last_quizzed` > 14 days old.
- End-of-session: overall score /100, per-lesson mastery updates (rule in vault-schema.md), quiz log note, and **new cards for every failed item** (which FSRS then drills to stability).

## Problem practice rules (used by /problems, /learn)

Problem-solving is a separate skill from recall and gets separate training:

- **Guidance fading:** sets progress worked-example completion (★) → scaffolded (★★) → fully independent (★★★), ending with one **transfer problem** in an unfamiliar context. Novices learn most from studying worked examples; experts from solving — fade accordingly as mastery rises.
- **Spaced re-attempts:** a failed problem (or one solved only with a hint) returns after 3–7 days, not the same day — re-solving after partial forgetting is what builds durable skill. A second consecutive fail → longer delay + directed re-reading first.
- **Interleave problem types** within a session (mixed sets outperform blocked sets robustly in the math/physics literature), and never label problems with the technique they need — diagnosing *which* method applies is most of the skill.
- **Attempt before solution, always.** Solutions live in folded callouts; the grading session never reveals one before an attempt or an explicit "show me".

## Dual coding rules (used by /learn, /ingest)

Pair words with structure-showing visuals — verbal + visual encoding beats either alone:

- Use a **Mermaid diagram** whenever the shape of the idea is the content: pathways, cycles, hierarchies, decision logic, state transitions, experimental setups. Obsidian renders ` ```mermaid ` natively; no images needed.
- Use **matplotlib-generated SVGs** (into the course `assets/` folder) for anything quantitative — real curves from real computations; never fabricate data or draw qualitative axes a computation could make exact.
- A diagram must *earn its place* by showing relations prose states poorly; decorative visuals are noise. Refer to the diagram from the prose ("note the branch at…") so the two encodings connect.

## Session/scheduling rules (used by /study)

- Daily order: (1) due-card review in Obsidian first — it's the highest-value 15 minutes; (2) retrieval work: due problem re-attempts and quizzes on weak/stale material; (3) only then new material (next lesson or ingest chunk). Pending Inbox items are processed at session start so open questions become knowledge instead of aging.
- Stale = `last_quizzed` older than 21 days with `mastery < 85` → requeue for quizzing.
- Respect `target_date`: as an exam approaches, shift the mix from new material toward cumulative interleaved quizzing (successive relearning).
- Keep sessions honest: the study log records planned vs. actually done; unfinished items carry over rather than silently vanish.
