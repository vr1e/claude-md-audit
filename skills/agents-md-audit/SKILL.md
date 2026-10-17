---
name: agents-md-audit
description: Audit, write, or migrate a repo's AGENTS.md / CLAUDE.md against its codebase. Use when asked to audit an existing one, write one for a repo that has none, move CLAUDE.md content into AGENTS.md, or work out why Claude ignores or doesn't load them.
allowed-tools: Read, Glob, Grep, Bash, Edit, Write
---

# AGENTS.md Improver

Audit and improve agent context files so they are **slim, verified, and worth their context cost**.

## File layout this skill enforces

- **`AGENTS.md`** is the canonical context file. All project guidance lives here. It is the one file every coding agent reads — Cursor, Codex, and Claude Code, which since v2.1.277 reads `AGENTS.md`, `.claude/AGENTS.md`, and nested `AGENTS.md` files directly when the project has no `CLAUDE.md`.
- **`CLAUDE.md` exists only when there is Claude-specific guidance** (skills, hooks, permissions, plan-mode rules). When it exists it must start with `@AGENTS.md`: Claude Code stops reading `AGENTS.md` on its own the moment a `CLAUDE.md`, `.claude/CLAUDE.md`, or `CLAUDE.local.md` is present in or above the working directory, so the import is what keeps the shared file loaded.

  ```markdown
  @AGENTS.md

  ## Claude Code

  - <Claude-specific instruction>
  ```

- **A bare pointer** — a `CLAUDE.md` holding nothing but `@AGENTS.md` — is a leftover from before native support. It is harmless (Claude never reads `AGENTS.md` twice) but it is a file kept alive for no gain, so propose removing it unless one of the keep conditions in [references/loading.md](references/loading.md) holds.

  Read [references/loading.md](references/loading.md) before choosing a layout action: which file turns Claude Code's reading of AGENTS.md on or off, nested and parent-directory CLAUDE.md files, imports, and `.claude/rules/`. Apply this whatever the starting state:

  | Starting state | Action |
  |---|---|
  | AGENTS.md only | Audit it. Create nothing else. |
  | AGENTS.md + bare pointer CLAUDE.md | Audit AGENTS.md; propose deleting the pointer (check the keep conditions first) |
  | CLAUDE.md is a symlink to AGENTS.md | Audit AGENTS.md and edit it, never the link. Leaving or deleting the link are both fine; replace it with an `@AGENTS.md` import when anyone works on Windows |
  | CLAUDE.md tells Claude in prose to read AGENTS.md | Delete it, or swap the sentence for `@AGENTS.md` when Claude-specific lines remain — prose loads AGENTS.md only if Claude decides to open it |
  | Full CLAUDE.md only | Migrate content → AGENTS.md. Delete CLAUDE.md, or reduce it to `@AGENTS.md` + the Claude-specific residue if there is any |
  | Both, duplicated | AGENTS.md becomes canonical; CLAUDE.md is deleted or reduced the same way |
  | Neither | Build a slim AGENTS.md ([references/templates.md](references/templates.md)) from *verified traps actually found*. If the repo has no traps, a near-empty file (one-line description + package manager if non-npm + non-obvious build command) is the correct output — never pad to look complete. |

  Whatever the row, also:

  - **CLAUDE.md above the repo** — report it: it turns off AGENTS.md reading for this repo. A personal workspace file (`~/Dev/CLAUDE.md`) moves to `~/.claude/CLAUDE.md`; when it has to stay (a parent repo this one is checked out inside), this repo keeps a CLAUDE.md that imports AGENTS.md.
  - **Nested CLAUDE.md** — apply the table to that directory too, and check the monorepo rules in loading.md.
  - **`SessionStart` hook that prints AGENTS.md** — propose removing it; it now adds a second copy.

  Never maintain the same content in both files, and never create a CLAUDE.md that has nothing Claude-specific to say. State the layout action in the Phase 4 report, per directory holding a context file, even when it is "no change".

## Core principle

Every line is loaded into every session's context. A line earns its place only if **an agent would act differently (or wrongly) without it**, and only if it is **verified true against the current codebase**. A false claim is worse than no claim — an agent trusts AGENTS.md over its own reading of the code.

## Workflow

### Phase 1: Discovery

```bash
root=$(git rev-parse --show-toplevel 2>/dev/null || pwd)

# Context files at any depth in the repo, plus rules and files Claude Code never reads
find "$root" \( -name node_modules -o -name .git \) -prune -o \( -name AGENTS.md -o -name CLAUDE.md -o -name CLAUDE.local.md -o -name AGENTS.local.md -o -name AGENTS.override.md -o -name .agents -o -path '*/.claude/rules/*.md' \) -print 2>/dev/null | head -100

# CLAUDE.md files above the repo: any of these turns off AGENTS.md reading
d=$(dirname "$root")
while [ "$d" != / ]; do
  for f in "$d/CLAUDE.md" "$d/CLAUDE.local.md" "$d/.claude/CLAUDE.md"; do
    [ -f "$f" ] && [ "$f" != "$HOME/.claude/CLAUDE.md" ] && echo "$f"
  done
  d=$(dirname "$d")
done

# Hooks that print AGENTS.md
grep -l 'AGENTS.md' "$root/.claude/settings.json" "$root/.claude/settings.local.json" ~/.claude/settings.json 2>/dev/null
```

Note the layout per directory that holds a context file (AGENTS.md only, + bare pointer, symlink, prose pointer, CLAUDE.md only, both, neither). `.claude/AGENTS.md` and `.claude/CLAUDE.md` count the same as their root counterparts. `.claude/rules/` files are part of the audit: rules without `paths:` frontmatter load every session like the root file. `AGENTS.local.md`, `AGENTS.override.md`, and `.agents/` reach other agents only; report any rule there that Claude also needs.

### Phase 2: Verify every claim

This is the most important phase. **Do not score or edit anything before doing this.** For each factual claim in the file:

1. **Open the file/config the claim is about.** "Uses Preact via next.config.ts aliasing" is checked in `next.config.ts`, where the aliasing may turn out commented out. A dependency in `package.json` doesn't prove it is used; a config file existing doesn't prove its setting is active.
2. **Run or trace commands.** Check that documented commands exist in `package.json`/`Makefile` and that descriptions of what they chain together are accurate.
3. **Check paths.** Every referenced file/directory must exist, and every `@` import must resolve (rules in loading.md). File paths are the fastest-rotting content in these files — treat every path as a mandatory existence check, and prefer keeping path references few.

Mark each claim verified or FALSE. A claim no file could confirm (process, intent, posture) is FILLER.

Verification often surfaces discrepancies in the code itself — a config launching a path the build doesn't produce, a workflow that never fires. These aren't AGENTS.md content, but note them for the report: they're free value from the audit, and the user will want to know.

### Phase 3: Classify every line, then check the whole file

Put every line in exactly one bucket: the first test it fails, in this order. A line that passes all four is EARNED.

1. **FALSE** — fails the *verification test*: the file it's about, opened in Phase 2, says otherwise. Fix first: correct it or delete it.
2. **FILLER** — fails the *behavior test*: an agent would act the same without it. Generic advice ("write tests", "use meaningful names"), posture addressed to auditors rather than agents ("protection against OWASP Top 10", "regular penetration testing"), one-off history, and in-flight state that will soon be false ("migrating to the new router", "current focus: checkout"). Delete it, with three exceptions:
   - a posture claim backed by real enforcement becomes a pointer to it ("input sanitization: `src/helper/sanitize.ts`")
   - a vague workflow line with a concrete check behind it becomes that check ("test your changes" → "`npm run lint` passes before finishing")
   - inside a CLAUDE.md, a maintainer note can stay as a block-level HTML comment, which Claude Code strips before loading
3. **DERIVABLE** — fails the *derivability test*: one `ls` or one Read of an obvious file (`package.json`, `README.md`, a config) gives it. Command lists restating scripts 1:1, tech-stack lists, content duplicated from README, directory maps. **Delete a directory map whole rather than slimming it** — the common failure is trimming a 10-line tree to 6 lines when the correct edit is zero lines; a directory earns a line only when its name misleads, and then it is a trap. Keep only the residue the source can't show: what an umbrella script chains, which check CI gates on, required env vars, which scripts never terminate (examples in [references/update-guidelines.md](references/update-guidelines.md)).
4. **RELOCATABLE** — fails the *globality test*: true and needed, but not in most sessions, or better held elsewhere. Move it — deleting it loses guidance someone needs:
   - one domain or one directory (TypeScript conventions, testing patterns, one package's quirks) → nested `AGENTS.md`, `docs/<DOMAIN>.md`, or a skill, per [references/templates.md](references/templates.md)
   - personal (sandbox URLs, `/Users/…` paths, "I prefer…") → the user's `~/.claude/CLAUDE.md` or auto memory, which load alongside AGENTS.md
   - a mechanical always/never rule (no edits in `src/generated/`, no committed `.env`, format after every edit) → the tooling that enforces it: a lint rule, pre-commit hook, or CI check for every agent and human; a Claude Code hook or `permissions.deny` for Claude alone. The prose line goes once the enforcement exists.
5. **EARNED** — verified, behavior-changing, not derivable, and relevant to most sessions: naming traps, conventions the code can't show, gotchas that cost a debugging session, environment facts, repository etiquette (branch naming, commit and PR format — every session that commits needs it, and no file path would load it on demand). Commit and PR rules compete with Claude Code's built-in git instructions; mention the `includeGitInstructions` and `attribution` settings next to them.

Then check the file as a whole, together with every file that loads beside it (nested files, `.claude/rules/`, the user's `~/.claude/CLAUDE.md`):

- **Contradictions** — two instructions that conflict, typically accumulated by different developers over time. Flag each pair and ask the user which to keep; never leave both.
- **Pointers** — the wording of a breadcrumb decides whether the agent opens the doc. Each one opens with the task that should send the agent there and says what it will find: "Before writing or changing tests, read `docs/TESTING.md` (fixtures, the DB mock, naming)", where "For testing, see docs/TESTING.md" names only a topic. A pointer that matters gets sharper wording, not capitals. The path goes in plain text or backticks — an `@` import loads the doc every session (loading.md).
- **Prohibitions** — a line that only bans ("don't call fetch directly") puts the banned thing in front of the agent. Rewrite it as the target ("API calls go through `src/api/apiClient.ts`"), keeping a ban only as a guardrail beside its target.
- **Emphasis** — IMPORTANT, MUST, NEVER, or all-caps on many lines. Emphasis works by contrast: keep it on the one line with a record of being skipped, and state the rest plainly.
- **Missing traps** — the highest-value content is usually absent. This check is done only when you have looked at each of these and either found the file covers it or added a line:
  - names that occur in more than one place with different roles (`pages/` vs `src/pages/`), and renamed entry points ("formerly `middleware.ts`")
  - root scripts that chain others, and scripts that never terminate
  - CI workflows that deploy or publish — quote their exact branch and path filters from the workflow file; a paraphrase on the scariest line in the file is as bad as an unverified claim
  - workspaces a root test script fans out to — which actually have tests, and which mocks or aliases the suite needs to run at all (the runner and framework stay out; they're one Read away)

### Phase 4: Report

Output the report **before** making any edits:

```
## AGENTS.md Audit — <path>

Verdict: <Good | Needs work | Rebuild> — <one line why>

### FALSE (fix first)
- <claim> — <what the code actually shows, with file:line>

### FILLER and DERIVABLE (cut)
- <section or lines> — <the test it fails; any residue worth keeping>

### RELOCATABLE (move)
- <lines> → <nested AGENTS.md / docs/<DOMAIN>.md / skill / ~/.claude/CLAUDE.md / lint rule / hook> — <the domain, directory, or person it's scoped to>

### Rewrites (pointers, prohibitions, emphasis)
- <current line> → <proposed line>

### Contradictions (user must pick)
- <instruction A> vs <instruction B>

### Missing traps (add)
- <trap/gotcha> — <where you found it>

### Possible real bugs noticed while verifying (FYI — not AGENTS.md content)
- <discrepancy in the code/config itself, e.g. a process manager launching a path the build doesn't produce>

### EARNED (keep)
- <the lines that earn their place>

### Structure
- <layout action per the table above, per directory — always state one, e.g. "AGENTS.md only, nothing to change", "delete bare pointer CLAUDE.md", "migrate CLAUDE.md → AGENTS.md and delete it">

Estimated size: <current lines, counting @-imported files and rules without paths:> → <proposed lines>
```

Verdicts:

- **Good** — mostly EARNED, nothing FALSE, ≲60 lines. Keep, minor trims.
- **Needs work** — anything FALSE, an unresolved contradiction, or more than a third FILLER/DERIVABLE/RELOCATABLE. A file over ~80 lines almost always lands here. Targeted edits.
- **Rebuild** — template boilerplate never customized, or more filler than signal. A fresh slim AGENTS.md from what the audit found.

### Phase 5: Apply

After user approval:

1. Check `git status` first — if the context files have uncommitted changes, suggest stashing (or committing) them before editing so the user can review the audit as a clean diff (and revert it wholesale if needed).
2. In AGENTS.md: fix FALSE lines, cut FILLER and DERIVABLE, apply the rewrites, add missing traps.
3. Move RELOCATABLE lines to their destinations ([references/templates.md](references/templates.md) has the destinations, the relocated-doc template, and the consolidation rules). Where the content won't load on its own, leave a breadcrumb that passes the pointer check.
4. Apply the layout action from the table above. If a CLAUDE.md survives, its first line is `@AGENTS.md` — otherwise Claude Code silently stops reading AGENTS.md.
5. **Final derivability sweep**: reread the proposed AGENTS.md section by section and delete any survivor an agent could get from one `ls` or one Read of `package.json`/`README.md` (directory maps, test-stack recitals, "deployed on X" lines). Filler tends to survive the first pass in slimmed-down form.
6. Run the validation checklist in [references/update-guidelines.md](references/update-guidelines.md), then show the resulting files — AGENTS.md should usually be *shorter* than before. If your edit made it longer without adding trap/gotcha content, reconsider.
7. If the layout changed, tell the user how to confirm what loads in their next session (see "Confirming the result" in [references/loading.md](references/loading.md)).
