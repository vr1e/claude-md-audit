# Templates

## CLAUDE.md — only when there is Claude-specific guidance

Claude Code reads `AGENTS.md` directly (v2.1.277+), so the default layout has no `CLAUDE.md` at all. Create one only for guidance that is Claude-specific — skills, hooks, permissions, plan-mode rules — and start it with the import, because as soon as a `CLAUDE.md` exists Claude Code reads it *instead of* `AGENTS.md`:

```markdown
@AGENTS.md

## Claude Code

- <Claude-specific instruction, e.g. skill/hook/permission guidance>
```

A `CLAUDE.md` that would hold only `@AGENTS.md` should not be created. If one already exists it is a deletion candidate — see the keep conditions in SKILL.md.

## AGENTS.md — slim template

Every section is optional. The best AGENTS.md files are 20–60 lines. When in doubt, leave a section out — the agent can read the code.

```markdown
# <Project Name>

<One line: what this is. Skip the tech-stack list — it's in package.json.>

## Commands (non-obvious only)

- `<umbrella command>` — chains <X → Y → Z>; CI gates on this, run before finishing
- <gotcha: required env var, "use test:watch locally but CI runs test", command that isn't a script>

## Structure traps

- <two similarly-named dirs with different roles, e.g. `pages/` = routes, `src/pages/` = view components>
- <"formerly named X" notes, misleading names, entry point that isn't where you'd expect>

## Conventions the code can't tell you

- <pattern with a why, e.g. "SVGs: import with `?url` for URLs, plain import = SVGR component">
- <ordering constraint, init dependency>

## Gotchas

- <thing that will bite an agent, verified against the code>

## Deeper guidance

- For <domain> conventions, see `docs/<DOMAIN>.md`
```

Breadcrumbs are one conversational line each — no "ALWAYS read", no all-caps forcing. Referenced docs load only when relevant and can reference deeper docs in turn.

**A near-empty AGENTS.md is a valid outcome.** If the repo has no traps, the correct file is just the one-line description, the package manager (if not npm), and any non-standard build command. Never pad a file to look complete — that is how auto-generated `/init` boilerplate is born.

### What deliberately has NO section here

- **Full command tables** mirroring `package.json` scripts — the agent reads `package.json` and treats it as the source of truth; a mirror only adds drift risk.
- **Tech stack lists** — derivable from `package.json` + lockfile.
- **Directory maps / architecture trees** — not even slimmed-down ones; `ls` is cheaper and never stale. A directory earns a line only when its name misleads, and that line goes under "Structure traps".
- **Security/quality posture statements** — only enforced, code-locatable facts belong (e.g. "input validation lives in `src/helper/sanitize.ts`"), never intentions.

## Relocated docs — `docs/<DOMAIN>.md`

Where relocated scoped content lands. Each doc is self-contained for its topic; prefer these names when they fit, so audits across repos produce a consistent set: `TESTING.md`, `TYPESCRIPT.md`, `CODE-STYLE.md`, `GIT-WORKFLOW.md`, `API-DESIGN.md`, `ARCHITECTURE.md`.

```markdown
# <Topic> Guidelines

<One line: when these apply — e.g. "Applies when writing or changing tests.">

## Rules

- Specific, actionable instruction
- <another — same bar as the root: verified and non-derivable; relocation is not a place to park filler>

## Examples

<Only when a rule is easier to show than state — a Good/Avoid pair. Omit otherwise.>
```

**Consolidate, don't fragment.** A relocated doc under ~10 lines usually belongs merged into a sibling, and more than ~6–8 relocated docs means the split is too granular — every extra doc is another breadcrumb taxing the root and another file to keep alive.

## Monorepo

Root AGENTS.md: cross-package traps and the one command that runs everything. Per-package AGENTS.md only when a package has traps of its own — not one per package by default.

Nested AGENTS.md files **merge with the root** in the agent's context: never repeat root content at package level, never link back up to the root ("see root AGENTS.md" — it's already loaded), and keep each level scoped to what's relevant there. Domain-scoped rules (per-language conventions, testing patterns) still belong in referenced docs, not in either level.
