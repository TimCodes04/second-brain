---
name: project
description: Learn by building — scoped computational mini-projects (simulate, verify, fit, derive) grounded in recently learned material; Claude scaffolds, the user writes the part that teaches.
argument-hint: "[course-slug | lesson | topic] [--scope 1h|3h]"
---

# /project [<course-slug | lesson | topic>] [--scope 1h|3h]

Applied encoding: computing a thing cements it better than re-reading it. Read `docs/pedagogy.md` and `docs/vault-schema.md` (project schema) first. Vault: `<vault>`.

## 1. Propose

Offer 2–3 project options grounded in recently read/quizzed material, each with: the goal, what understanding it cements, estimated scope (default ~1h; `--scope 3h` for deeper). Favor: simulate a phenomenon from a lesson, numerically verify a formula or claim, fit a model to generated/real data, or a computational derivation check. Read `Profile.md` — a project that touches the user's actual work is worth proposing when the connection is genuine. The user picks or names their own.

## 2. Set up

Create `projects/<kebab-name>/` in the vault: a project note (`type: project` frontmatter per vault-schema: `course_slug`, `lessons` links, `status: active`, `summary`) stating the goal, the plan, and success criteria — plus the code file(s) (`python3`, numpy/matplotlib; if imports fail, tell the user what to `pip install` rather than working around it).

## 3. Build — scaffold, don't solve

The division of labor is the pedagogy:
- **Agent writes:** skeleton, I/O, plotting boilerplate, test harness — the plumbing.
- **User writes:** the core physics/math/logic — the part that teaches. Mark it with `# TODO(you): …` and explain what each block must do.
- Review their code *with* them: run it, interpret failures together, hint before fixing; take over a step only when they explicitly ask.

## 4. Wrap up

Run the final version; save figures into the project folder and embed them in the project note. Prompt the user for 3–5 "what this taught me" bullets (their words) and record them. Set `status: done`, link the project from the source lesson's and concepts' `## Related`, and mint 0–2 cards if durable insights emerged. Unfinished is fine — `status: active` projects get surfaced by `/study` and `/brief` as an open thread.
