# Obsidian setup checklist

Everything here happens in the Obsidian UI and takes about 10 minutes. The agent never touches `.obsidian/`. `/setup` walks you through this list, and `/setup --check` verifies it afterwards.

## 1. Open the vault

Obsidian → **Open folder as vault** → choose the vault folder you gave `/setup`. Bases needs **Obsidian 1.9.10 or newer**; check under Settings → About.

## 2. Core plugins

Settings → **Core plugins** → make sure these are on:

- **Bases** powers the dashboards on `Home.md`.
- **Templates**: then go to Settings → Templates → *Template folder location* and enter `templates`.

## 3. Install the Spaced Repetition plugin

1. Settings → **Community plugins** → turn off Restricted mode if prompted.
2. **Browse** → search **"Spaced Repetition"** (by Stephen Mwangi, id `obsidian-spaced-repetition`) → Install → Enable.

## 4. Configure it (Settings → Spaced Repetition)

| Setting | Value | Why |
|---|---|---|
| **Flashcard tags** | `#flashcards` | Decks come from hierarchical tags (`#flashcards/<course>`) |
| **Convert ==highlights== to clozes** | On | The cloze card format the skills write |
| **Save scheduling comment on the same line as the flashcard's last line?** | On | Keeps card blocks compact and easy to grep |
| **Folders to ignore** | `templates` | Template skeletons must never enter the review queue |
| **Algorithm** | **FSRS** (recommended) or the default *OSR's variant of SM-2* | FSRS schedules ~20–30% fewer reviews at equal retention |
| Separators | leave the defaults (`::` inline, `?` multiline) | The skills write exactly these |

About FSRS: the plugin marks it as newer and shows a data-loss warning when you switch. Switch **before** you have review history, or commit the vault to git first. Both algorithms work with this system, because the skills parse both scheduling formats. Leave the FSRS parameters at their defaults (target retention ≈ 0.9) and revisit them only after about a month of real reviews.

## 5. Dashboards sanity check

Open `Home.md`. The embedded Courses / Lessons / Quiz Results views should render (empty at first). If a view's sort or columns look wrong, fix it in the view's menu. Obsidian writes the fix back into the `.base` file, which is the canonical way to correct syntax drift.

## 6. Reviewing cards

Command palette → **"Spaced Repetition: Review flashcards from all notes"** (or the deck icon in the left ribbon). This works on desktop and mobile. Daily review is the highest-value 15 minutes of the whole system.

## 7. Optional: phone access and backup

- **Sync to your phone**: [Obsidian Sync](https://obsidian.md/sync) (paid, end-to-end encrypted, most reliable), iCloud Drive (Apple devices only; the vault must live in `iCloud Drive/Obsidian/`), or the community *Git* plugin (free; mobile support works but is less reliable).
- **Version history**: `/setup` can `git init` the vault. The skills then commit after every session, and `git -C <vault> log` / `git -C <vault> checkout <commit> -- <file>` restores any earlier state. If you add a remote, keep it **private**: the vault holds your notes, quiz history, and `Profile.md`.
