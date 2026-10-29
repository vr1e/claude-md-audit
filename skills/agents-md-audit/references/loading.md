# How Claude Code loads context files

Per the Claude Code memory docs as of v2.1.285. Read before choosing a layout action, and whenever a CLAUDE.md, an `@` import, a symlink, `.claude/rules/`, or a nested file is involved.

## The AGENTS.md switch

Claude Code (v2.1.277+) reads `AGENTS.md` and `.claude/AGENTS.md` directly only while the switch is **on**: no `CLAUDE.md`, `.claude/CLAUDE.md`, or `CLAUDE.local.md` in the launch directory or any directory above it. One such file anywhere on that path turns it **off**, and Claude reads CLAUDE.md files only.

- **Never flip the switch**, and keep loading alongside AGENTS.md: `~/.claude/CLAUDE.md`, the organisation's managed CLAUDE.md, `.claude/rules/` files.
- **Nested files, switch on**: a subdirectory's AGENTS.md loads when Claude reads a file there, unless that subdirectory has a CLAUDE.md of its own.
- **Switch off**: no AGENTS.md loads directly, root or nested. Each reaches Claude only through an `@AGENTS.md` import in a CLAUDE.md beside it.
- **Above the repo**: a CLAUDE.md in a parent directory (a `~/Dev/` workspace, a parent repo this one is checked out inside) turns the switch off for every repo below it. When it is the user's personal file, its content belongs in `~/.claude/CLAUDE.md`.
- **Monorepos**: a nested `packages/api/CLAUDE.md` hides `packages/api/AGENTS.md` unless it imports it, and anyone launching Claude from `packages/api/` turns the switch off for the whole session, dropping the root AGENTS.md. So once any CLAUDE.md is needed inside the repo, every AGENTS.md on a launch path needs a CLAUDE.md beside it that imports it.
- **Never read by Claude Code**: `AGENTS.local.md`, `AGENTS.override.md`, anything under `.agents/`. Their content reaches other agents only.

## When a bare `@AGENTS.md` pointer stays

A bare pointer is redundant once every session that opens the repo reads AGENTS.md directly. The file is shared, so keep it when any of these holds for the user *or a teammate*:

- Claude Code older than v2.1.277, or older than v2.1.281 on Amazon Bedrock or with telemetry disabled
- The built-in AGENTS.md plugin is disabled through `/plugin`
- **Project instructions** is `claude-md` or `managed-only`: `options.instructionFiles` under `pluginConfigs."cc-plugin-agents-md@builtin"` (before v2.1.285: `agents-md@builtin`) in `~/.claude/settings.json`
- A `CLAUDE.local.md` is in use: it turns the switch off, and the pointer brings AGENTS.md back
- The team relies on something that fires only for CLAUDE.md: `InstructionsLoaded` hooks, `--add-dir` with `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD`, or an AGENTS.md import of a file outside the repo (it loads only if external imports were already approved, and Claude never prompts for it)

Settle the user's side yourself (`claude --version`, `~/.claude/settings.json`). Teammates you can't check: list their conditions in the report as the reason to object, and delete the pointer with the rest of the approved edits unless the user names one that holds. Never leave it in place just because nobody answered.

## Imports

- `@path` outside a code span or fenced block imports the file **at launch**. Imports organise a file; they never cut its context cost, so count imported files in the size estimate.
- Breadcrumbs name the path in plain text or backticks (`` `docs/TESTING.md` ``). An `@docs/TESTING.md` breadcrumb loads the doc every session and undoes the relocation.
- Paths resolve relative to the importing file, nest at most four hops, and need a backslash before each space (`@Design\ Docs/x.md`); a quoted path is not imported at all. Check every import resolves.

## Other mechanics

- **`.claude/rules/`**: a rule without `paths:` frontmatter loads every session with the priority of `.claude/CLAUDE.md`, so it counts toward the root budget. With `paths:` it loads when Claude reads or edits a matching file, and loads again on the next match after `/compact`. Only Claude reads these files.
- **HTML comments**: block-level `<!-- … -->` in a CLAUDE.md is stripped before Claude sees it, a free home for maintainer notes. The docs say this of CLAUDE.md only; treat comments in AGENTS.md as loaded, and other agents read them too.
- **Symlinked CLAUDE.md → AGENTS.md**: read once, through the link. Edit and Write refuse to write through a symlink, so edit AGENTS.md. Git checks a committed symlink out as a one-line text file on Windows clones without `core.symlinks`; when anyone works on Windows, use the `@AGENTS.md` import.
- **Prose pointer** ("read AGENTS.md first"): AGENTS.md loads only if Claude decides to open it.
- **`SessionStart` hook printing AGENTS.md**: once Claude reads AGENTS.md directly, the hook adds a second copy.

## Confirming the result

After a layout change, tell the user how to check it in their next session: with no CLAUDE.md, an interactive session prints `no CLAUDE.md found; AGENTS.md loaded: <path>`; `/memory` lists every loaded file (AGENTS.md included from v2.1.280); `/context` lists CLAUDE.md and rules files under **Memory files**.
