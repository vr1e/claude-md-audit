# Templates

## CLAUDE.md — only when there is Claude-specific guidance

Claude Code reads `AGENTS.md` directly (v2.1.277+), so the default layout has no `CLAUDE.md` at all. Create one only for guidance that is Claude-specific — skills, hooks, permissions, plan-mode rules — and start it with the import, because as soon as a `CLAUDE.md` exists Claude Code reads it *instead of* `AGENTS.md`:

```markdown
@AGENTS.md

## Claude Code

- <Claude-specific instruction, e.g. skill/hook/permission guidance>
```

A `CLAUDE.md` that would hold only `@AGENTS.md` should not be created. If one already exists, delete it unless a keep condition in loading.md holds.

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
- <branch naming, commit or PR format, when the repo has a convention>

## Gotchas

- <thing that will bite an agent, verified against the code>

## Deeper guidance

- Before <task that needs it>, read `docs/<DOMAIN>.md` (<what's inside>)
```

Each breadcrumb is one line that passes the pointer check in SKILL.md Phase 3. Referenced docs load only when relevant and can reference deeper docs in turn.

**A near-empty AGENTS.md is a valid outcome.** If the repo has no traps, the correct file is just the one-line description, the package manager (if not npm), and any non-standard build command. Never pad a file to look complete — that is how auto-generated `/init` boilerplate is born.

The template has no section for command tables, tech stacks, directory maps, or security posture: those are DERIVABLE or FILLER (SKILL.md Phase 3). A directory whose name misleads goes under "Structure traps".

## Where relocated content goes

Pick the destination by the shape of the scope:

- **One directory** (a package, `src/api/`) → a nested `AGENTS.md` in that directory. Claude Code loads it when it reads a file there, other agents read nested AGENTS.md too, and the root needs no breadcrumb. This holds only while no CLAUDE.md turns off AGENTS.md reading (loading.md).
- **A cross-cutting domain** (testing patterns, TS conventions) → `docs/<DOMAIN>.md` plus a one-line breadcrumb in the root.
- **A multi-step procedure** (release, data migration) → a skill.
- **`.claude/rules/<topic>.md` with `paths:`** → only in a repo worked on with Claude Code alone, for a narrow glob that cuts across directories (`**/*.test.ts`, `migrations/**`). Other agents never see it, and a broad glob (`**/*.ts` in a TypeScript repo) loads in nearly every session, saving nothing.

## Relocated docs — `docs/<DOMAIN>.md`

Each doc is self-contained for its topic; prefer these names when they fit, so audits across repos produce a consistent set: `TESTING.md`, `TYPESCRIPT.md`, `CODE-STYLE.md`, `API-DESIGN.md`, `ARCHITECTURE.md`.

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

## Nested AGENTS.md and monorepos

A directory gets its own AGENTS.md only when it has traps or rules of its own — not one per package by default. In a monorepo the root holds cross-package traps and the one command that runs everything.

Nested AGENTS.md files **merge with the root** in the agent's context: never repeat root content at a nested level, never link back up to the root ("see root AGENTS.md" — it's already loaded), and keep each level scoped to what's relevant there. Cross-cutting domain rules (per-language conventions, testing patterns) still go to `docs/<DOMAIN>.md`, not to either level.

Keep CLAUDE.md files out of subdirectories. A nested CLAUDE.md hides its directory's AGENTS.md unless it imports it, and launching Claude from that directory drops the root AGENTS.md; once one exists, follow the monorepo rule in loading.md.
