---
name: agents-md-audit
description: Audit and improve AGENTS.md / CLAUDE.md files. Use when user asks to check, audit, update, improve, or fix AGENTS.md or CLAUDE.md files. Verifies every claim against the actual codebase, strips content derivable from package.json/README, relocates domain-scoped rules, and keeps the file slim to preserve context. Also use when the user mentions "AGENTS.md maintenance", "CLAUDE.md maintenance", or "project memory optimization".
allowed-tools: Read, Glob, Grep, Bash, Edit, Write
---

# AGENTS.md Improver

Audit and improve agent context files so they are **slim, verified, and worth their context cost**.

## File layout this skill enforces

- **`AGENTS.md`** is the canonical context file. All project guidance lives here.
- **`CLAUDE.md`** is a thin pointer that imports it. This isn't just tidiness: Claude Code reads `CLAUDE.md`, not `AGENTS.md` — the pointer is what makes AGENTS.md load at all (and it's the layout the official memory docs recommend for repos shared with other coding agents):

  ```markdown
  # CLAUDE.md

  @AGENTS.md

  ## Claude Code

  (only Claude-specific guidance, if any — otherwise omit this section)
  ```

  Enforce this layout in every audit, whatever the starting state:

  | Starting state | Action |
  |---|---|
  | AGENTS.md only | Audit it, **and create the pointer CLAUDE.md** |
  | Full CLAUDE.md only | Migrate content → AGENTS.md, replace CLAUDE.md with the pointer |
  | Both, duplicated | AGENTS.md becomes canonical, CLAUDE.md becomes the pointer |
  | Neither | Build a slim AGENTS.md from *verified traps actually found*, plus the pointer. If the repo has no traps, a near-empty file (one-line description + package manager if non-npm + non-obvious build command) is the correct output — never pad to look complete. |

  Never maintain the same content in both files. Creating the pointer CLAUDE.md is part of the standard output, not an optional extra — include it in the Phase 4 report.

## Core principle

Every line is loaded into every session's context. A line earns its place only if **an agent would act differently (or wrongly) without it**, and only if it is **verified true against the current codebase**. A false claim is worse than no claim — an agent trusts AGENTS.md over its own reading of the code.

## Workflow

### Phase 1: Discovery

```bash
find . -maxdepth 3 \( -name "AGENTS.md" -o -name "CLAUDE.md" -o -name "CLAUDE.local.md" \) -not -path "*/node_modules/*" 2>/dev/null | head -50
```

Note which layout the repo uses (AGENTS.md + pointer, CLAUDE.md only, both, neither).

### Phase 2: Verify every claim

This is the most important phase. **Do not score or edit anything before doing this.** For each factual claim in the file:

1. **Open the file/config the claim is about.** A claim like "uses Preact via next.config.ts aliasing" must be checked in `next.config.ts` — dependencies listed in `package.json` do not prove a claim (the code may be commented out or unused).
2. **Run or trace commands.** Check that documented commands exist in `package.json`/`Makefile` and that descriptions of what they chain together are accurate.
3. **Check paths.** Every referenced file/directory must exist. File paths are the fastest-rotting content in these files — treat every path as a mandatory existence check, and prefer keeping path references few.

Classify each claim: **verified**, **false** (delete or correct — highest priority), or **unverifiable** (aspirational statements, process claims like "regular penetration testing" — delete).

Verification often surfaces discrepancies in the code itself — a config launching a path the build doesn't produce, a workflow that never fires. These aren't AGENTS.md content, but note them for the report: they're free value from the audit, and the user will want to know.

### Phase 3: Audit for context waste

Flag content in these categories (see [references/quality-criteria.md](references/quality-criteria.md)):

1. **Derivable mirrors** — command lists that restate `package.json` scripts 1:1, tech-stack lists derivable from `package.json`, content duplicated from README. These drift and add no signal. Keep only the *non-derivable* part (e.g. "`npm run lint` chains tsc → eslint → stylelint; CI gates on it").
   **Directory maps are in this category — delete the section, don't slim it.** `src/model/ — TypeScript types` is one `ls` away. A directory earns a line only if its name misleads or collides with another (and then it belongs under traps, not in a map). The common failure mode is trimming a 10-line tree to a 6-line tree; the correct edit is usually zero lines.
2. **Aspirational filler** — security/quality boilerplate describing intentions, not enforced reality ("protection against OWASP Top 10"). Delete unless it points to actual enforcement code.
3. **Generic advice** — "write tests", "use meaningful names". Delete.
4. **Scoped content to relocate** — true and useful, but only relevant to one domain (TypeScript conventions, testing patterns, API design, git workflow). Don't keep it in the root (it loads on every session) and don't delete it: move it to a referenced doc (`docs/TESTING.md`) or a skill, leaving a one-line breadcrumb ("For TS conventions, see docs/TYPESCRIPT.md"). Root lines must be relevant to *most* sessions.
5. **Contradictions** — instructions that conflict with each other, typically accumulated by different developers over time. Flag each pair and ask the user which to keep; never leave both.
6. **Missing traps** — the highest-value content is usually *absent*: naming collisions (two `pages/` directories), gotchas, ordering constraints, "formerly named X" notes. Scan the repo structure for confusable directories/files the file doesn't warn about. Two surfaces that reliably hide traps: **destructive workflow triggers** — for anything outward-facing (production deploy, publish), quote the exact branch + path filters from the workflow file, never a paraphrase ("root package files" vs the actual `package.json`/`package-lock.json` globs) — a loose paraphrase on the scariest line in the file is the same failure class as an unverified claim; and **test-surface asymmetries** — when a root script fans out over workspaces, note which workspaces actually have tests and any mocks/aliases the suite needs to run at all. The runner and framework themselves stay out — they're one Read away.

### Phase 4: Report

Output the report **before** making any edits:

```
## AGENTS.md Audit — <path>

### False or unverifiable claims (fix first)
- <claim> — <what the code actually shows, with file:line>

### Derivable / filler content (cut)
- <section or lines> — <why it adds no signal>

### Scoped content (relocate)
- <lines> → <destination doc/skill> — <which domain it's scoped to>

### Contradictions (user must pick)
- <instruction A> vs <instruction B>

### Missing high-value content (add)
- <trap/gotcha> — <where you found it>

### Possible real bugs noticed while verifying (FYI — not AGENTS.md content)
- <discrepancy in the code/config itself, e.g. a process manager launching a path the build doesn't produce>

### Verified keepers
- <the lines that earn their place>

### Structure
- <AGENTS.md/CLAUDE.md layout action per the table above — always state one, e.g. "create pointer CLAUDE.md">

Estimated size: <current lines> → <proposed lines>
```

### Phase 5: Apply

After user approval:

1. Check `git status` first — if the context files have uncommitted changes, suggest stashing (or committing) them before editing so the user can review the audit as a clean diff (and revert it wholesale if needed).
2. Fix false claims, cut filler, add missing traps in AGENTS.md.
3. Relocate scoped content to its destination docs/skills and replace it with one-line breadcrumbs — use the relocated-doc template and consolidation rules in [references/templates.md](references/templates.md).
4. Apply the layout action from the table above — including **creating the pointer CLAUDE.md when it doesn't exist**.
5. **Final derivability sweep**: reread the proposed AGENTS.md section by section and delete any survivor an agent could get from one `ls` or one Read of `package.json`/`README.md` (directory maps, test-stack recitals, "deployed on X" lines). Filler tends to survive the first pass in slimmed-down form.
6. Show the resulting files — AGENTS.md should usually be *shorter* than before. If your edit made it longer without adding trap/gotcha content, reconsider.

## Templates

See [references/templates.md](references/templates.md) for the slim AGENTS.md template and the CLAUDE.md pointer.

## Litmus tests for each line

- **Derivability test**: Could the agent get this in one cheap Read of an obvious file (`package.json`, `README.md`)? Then cut it — the obvious file is the source of truth and this copy will drift.
- **Behavior test**: Would an agent do something *wrong* without this line? If it would merely be uninformed-but-fine, cut it.
- **Globality test**: Does this apply to most sessions in this repo, or only to one domain (frontend, testing, CI)? If scoped, relocate it to a referenced doc/skill and leave a breadcrumb. The root bar is: **undiscoverable AND globally relevant**.
- **Verification test**: Did you personally confirm this against the code *in this audit*? If not, don't keep it.
- **Audience test**: Is this addressed to an agent editing code, or to a human auditor/stakeholder? Cut the latter.
