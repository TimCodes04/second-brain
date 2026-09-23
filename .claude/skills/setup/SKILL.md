---
name: setup
description: One-command install and health check — asks where your knowledge base (Obsidian vault) should live, writes config.env, scaffolds the vault (folders, Home, Inbox, Profile, dashboards, templates), and walks you through the Obsidian plugin settings. Safe to re-run; never overwrites your notes.
argument-hint: "[vault-path] [--check] [--sync-templates]"
disable-model-invocation: true
---

# /setup [<vault-path>] [--check] [--sync-templates]

Goal: a new user goes from `git clone` to a working system with this one command and one or two answers. Keep it fast and friendly: ask one question message, batch file operations so the user sees as few permission prompts as possible, and don't lecture.

**Idempotent:** creates only what is missing. Never overwrites, moves, or deletes an existing vault file (template files under `--sync-templates` are the only exception, and only after confirmation). Never writes into `.obsidian/` (CLAUDE.md hard rule 2). `<repo>` = `git rev-parse --show-toplevel`. Today = `date +%F`.

Modes: no flag → steps 1–6. `--check` → step 7 only (read-only). `--sync-templates` → step 8 only.

## 1. Collect the configuration (one message at most)

Read `<repo>/config.env.example`. **It defines what to ask:** every variable in it, together with its comment, is one setting, so variables added there later are picked up automatically. If `<repo>/config.env` exists, its values are the defaults.

- If `config.env` already has a valid `VAULT_PATH` and no path argument was given, show the current values and ask "keep these?". Then continue with step 3 (the scaffolding still fills in anything missing).
- Otherwise ask everything still unknown in **one** message, with a default for each so the user can reply "ok":
  - `VAULT_PATH` (required). Use the `<vault-path>` argument if one was given. Otherwise ask for the knowledge-base folder: an existing Obsidian vault, or a new folder to create (suggest `~/Documents/SecondBrain`).
  - Every other variable, with its default from the example and its purpose in a few words.
  - "Keep a version history of your vault with git? (recommended: yes)"

Validate before writing. Expand `~` to an absolute path. `VAULT_PATH` must not be inside `<repo>`, and must not be `/` or the bare home directory. If the folder exists, is not empty, and has no `.obsidian/`, confirm it's the right folder. A non-empty `SOURCES_PATH` must exist.

## 2. Write the configuration

- `<repo>/config.env`: a copy of `config.env.example` with the values filled in (absolute, quoted; comments kept).
- `<repo>/.claude/settings.local.json`: merge into any existing JSON and keep every existing key and entry. Add `VAULT_PATH` (and `SOURCES_PATH`, if set) to `permissions.additionalDirectories`. Add `Edit(//<VAULT_PATH without its leading slash>/.obsidian/**)` to `permissions.deny`, which enforces hard rule 2 at the permission layer.

Both files are gitignored. Never write these values into tracked files.

## 3. Scaffold the vault (batched)

Use **one** shell command so the user approves once:
- `mkdir -p` the folders `atlas/maps`, `atlas/bases`, `concepts`, `courses`, `sources`, `palace`, `projects`, `log/study`, `log/briefs`, `templates`.
- Copy without overwriting (`cp -n` or an equivalent): `<repo>/scaffold/*` into the vault (`Home.md`, `Inbox.md`, `Profile.md`, `palace/_Palace Guide.md`, `log/inbox-archive.md`, `atlas/bases/*.base`), `scaffold/gitignore` → `<vault>/.gitignore` (only if absent), and `<repo>/templates/*.md` → `<vault>/templates/`.
- Fill `{{YYYY-MM-DD}}` with today and `{{repo_path}}` with `<repo>`, **only in the files just created**. Never touch pre-existing files, or the placeholders inside `templates/`.

Report what was created and what already existed.

## 4. Version history (if the user said yes)

If `<vault>` is not a git repository yet: `git -C "<vault>" init`, then commit everything as `setup: scaffold vault`. Never add a remote or push. If asked, explain how to add one, and stress that it must be **private**.

## 5. Obsidian: the only manual part

Show this checklist compactly, details in `docs/setup.md`. You cannot change these settings, and must not try:

1. Obsidian → **Open folder as vault** → `<vault>`
2. Settings → Core plugins → enable **Bases** and **Templates** (template folder: `templates`)
3. Settings → Community plugins → Browse → install and enable **Spaced Repetition** (by Stephen Mwangi)
4. Spaced Repetition settings: flashcard tags `#flashcards` · convert ==highlights== to clozes **on** · scheduling comment on the same line **on** · folders to ignore `templates` · algorithm **FSRS**
5. Open `Home.md`: the dashboards should render

## 6. Done

End with a short summary, then:
- **Restart once:** `/exit`, then `claude`, so `config.env` and the vault permissions load. After that, `/setup --check` confirms everything.
- **First steps:** `/learn "<something you want to learn>"`, `/ingest <a PDF>` for existing material, jot questions into `Inbox.md` from any device, and `/study` daily.
- Offer a 2-minute `Profile.md` interview (current work & projects, interests, background), now or anytime. It's optional; answers are recorded in the user's own words.

## 7. `--check`: health check (read-only)

Report ✅/⚠️ for each item, with a one-line fix per ⚠️:
- `config.env` present; `VAULT_PATH` (and `SOURCES_PATH`, if set) exist; both are listed in `.claude/settings.local.json`; `config.env` has every variable that `config.env.example` defines (new ones appear after updates → re-run `/setup`).
- Required folders and scaffold files present in the vault.
- `atlas/bases/*.base` and `templates/*.md` match the repo copies. List the differing files; Obsidian re-serializes `.base` files, so compare the filters and views, not the formatting.
- Plugins, read-only from `.obsidian/`: `plugins/obsidian-spaced-repetition/manifest.json` exists and its major version is 1 (the scheduling format in `docs/vault-schema.md` was verified against v1.15.4). `core-plugins.json` has `bases` and `templates` enabled.
- Vault git state (repo? clean? remote?) and whether it matches `VAULT_GIT_PUSH`.
- Schema drift: courses whose `lessons_done` doesn't match their lessons' statuses, and concept notes with `mastery: 0` that no quiz log mentions (older vaults stored `0` instead of empty). Offer to fix these only on a yes.

## 8. `--sync-templates`

Diff `<repo>/templates/*.md` and `<repo>/scaffold/atlas/bases/*.base` against the vault copies, show which files differ and how, and overwrite the vault copies only after the user confirms. These are skeletons and dashboard definitions, not notes. Never touch other vault files here.

Finish every mode that changed the vault with the vault commit (CLAUDE.md conventions).
