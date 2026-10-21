# AGENTS.md — Clauderreich-DH (Kaiserreich for Darkest Hour)

Working notes for any agent session in this workspace. **This file is loaded
automatically at session start.** `MOD_STRUCTURE.md` (same directory, gitignored)
is the full structural reference and is the only source of truth for how the mod
loads — read the relevant section before changing anything.

> If `MOD_STRUCTURE.md` is missing from this directory, it has not been carried
> over to this clone. Say so rather than guessing at the mod's internals.

## What this is

A HOI2 **Darkest Hour** mod — the Kaiserreich scenario, forked. The engine
resolves every file by path relative to the game root, **mod first, then the
base game**: same path = replace, new path = add, folders merge and never
shadow each other. The scenario `scenarios/0_Kaiserreich.eug` is the root, and
vanilla `db/events.txt` is never loaded under Kaiserreich.

**The mod must keep booting.** A change that breaks the game is worse than no
change at all.

## Read before you touch anything

| If you are… | Read `MOD_STRUCTURE.md` §… |
|---|---|
| adding or editing an event | 6 (events), 6.6 (ID ranges), 12 A (recipe), 7.7 (manifests) |
| adding a country | 12 B (recipe), 5 (scenario data), 7 (db) |
| changing localisation | 8 (`config/`), 3.5 (whole-file replacement) |
| touching art, map or AI | 9, 11, 10 |
| balancing or tech trees | 7.4, 12 D–E |
| recording or committing a change | **13 — versioning & changelog workflow** |

## Hard constraints

- **A file on disk that is not listed in its manifest is silently ignored.** The
  manifests are `db/leaders.txt`, `db/ministers.txt`, `db/teams.txt` and
  `db/events kaiserreich.txt`. A new file always needs a manifest line.
- **Event IDs must be globally unique.** The team's allocation sheet is
  `events id range.xls` — binary, so it cannot be parsed here; open it manually.
- **Localisation is whole-file replacement, not a merge.** `config/*.csv` rows
  are semicolon-separated, `#` rows are ignored, the last cell of column A must
  be `#EOF`, and `# ; % ? " \` are illegal inside values.
- **`db/misc.txt` — the order of its variables cannot be changed.**
- **`db/policy_effects.csv` — do not add, remove or reorder any line or column.**
- **Never edit a `Changelog.txt` entry that has already been committed.** Add a
  new version instead.
- **`MOD_STRUCTURE.md` is gitignored and must stay out of the repo.**

## Making a change

1. Make the change.
2. Add a `Changelog.txt` entry under your own name — §13 has the version
   scheme and the exact entry format.
3. Update `MOD_STRUCTURE.md` if the change invalidates anything it states.
4. Verify (§13, "Verify before committing").
5. Commit as `v<version>: <one-line summary>`. **One version = one commit.**

## Versioning

Our versions are `<upstream version>.<increment>` — `1.9.3.1`, `1.9.3.2`, …
The increment is **per contributor**: each person keeps their own counter under
their own heading in `Changelog.txt`. The upstream baseline is Kaiserreich-DH
**1.9.3** (commit `793063d11`); our fork is `github.com/sp00nexe/Clauderreich-DH`,
branch `master`.

## Plumbing

- `git` is **not on PATH**. Use GitHub Desktop's bundled binary:
  `"C:\Users\sp00n\AppData\Local\GitHubDesktop\app-3.6.7\resources\app\git\cmd\git.exe"`.
- `core.autocrlf = true` and there is **no `.gitattributes`** — the working tree
  is CRLF, the index is LF. Write CRLF into the mod's text files.
- Mod root: `C:\Program Files (x86)\Steam\steamapps\common\Darkest Hour A HOI Game\Mods\Clauderreich-DH`.
  Vanilla game it overlays: `C:\Program Files (x86)\Steam\steamapps\common\Darkest Hour A HOI Game`.
- `MOD_STRUCTURE.md` is UTF-8 **without BOM** and **LF**-only. Rewrite it with
  `[System.IO.File]::ReadAllText` / `WriteAllText` and
  `New-Object System.Text.UTF8Encoding($false)` — PowerShell 5.1's
  `Get-Content` / `Set-Content` corrupt its non-ASCII characters.
- The engine writes `savedebug.txt` to the game root. It is the first place to
  look for event-registration and file-parse errors.

## Don't break the mod

Before committing, re-run the manifest coverage check and confirm no ID is
used by two files (§7.7 has the recipe). If you are not sure whether a change
is safe, say so instead of guessing.
