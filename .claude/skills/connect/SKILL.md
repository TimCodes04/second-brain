---
name: connect
description: Discover non-obvious, cross-domain connections between vault notes and write them as real wikilinks into both notes' Related sections — making hidden structure visible in the graph.
argument-hint: "[--course slug | --all]"
---

# /connect [--course <slug> | --all]

Cross-domain link discovery. Read `docs/vault-schema.md` first. Vault: `<vault>`.

1. **Harvest the corpus** (cheap, no embeddings): for every content note in scope (`--course` limits to that course's notes + all `concepts/`; default `--all` = every course/concept/MOC/reference/project note; skip `templates/`, card and problem files, and the system notes `Home.md`, `Inbox.md`, `log/inbox-archive.md`, `palace/_Palace Guide.md`), collect one line: `title | type | domains | summary` from frontmatter. Include `Profile.md` — connections to the user's *current work* are the most valuable kind. Also grep each note's existing wikilinks so already-linked pairs are excluded.
2. **Propose:** from the corpus, generate candidate pairs that are **non-obvious and cross-domain** — same mathematical structure, transferable technique, same phenomenon in different clothes (e.g. damped oscillator ↔ RLC circuit ↔ posterior width in inference). Each candidate: the pair + a one-sentence mechanism. Obvious within-topic pairs (already same course/section) don't count. Aim for quality: 3–10 candidates, ranked.
3. **Confirm in the terminal:** present the ranked list; the user picks (numbers, "all", or "none").
4. **Write:** for each accepted pair, append to BOTH notes' `## Related` sections: `- [[Other Note]] — <one-line mechanism>` (create the section if missing; no duplicates; bump `updated:`). These are real wikilinks → real edges in graph view.
5. **Optional bridge cards:** for striking connections, offer a card ("What do X and Y have in common structurally?") into `concepts/Inbox Cards.md` per the `/card` procedure.
6. Report: links written, and any cluster observations worth a future MOC section.
