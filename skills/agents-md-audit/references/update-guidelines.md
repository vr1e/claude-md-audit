# What earns a line

Shared by the audit (Phase 5) and `/revise-agents-md`. Whether a line belongs is decided by the five buckets in SKILL.md Phase 3; this file shows what EARNED lines look like, how to propose a change, and the checklist that closes one. The default direction of a change is **removal or relocation** — successful audits usually make the root file shorter.

## EARNED lines, by kind

### Traps and disambiguations

```markdown
- `pages/` = Next.js route files; `src/pages/` = the PascalCase view components they render
- Root entry point is `proxy.ts` (formerly `middleware.ts` — that file no longer exists)
```
Why: two things with the same name and different roles is exactly what sends an agent down the wrong path.

### Umbrella commands and CI gates

```markdown
- `npm run lint` chains tsc → eslint → stylelint; CI gates on it — run before finishing
```
Why: what a script *chains* isn't visible from the script list; which check gates CI isn't in package.json at all.

### Non-obvious conventions, with location

```markdown
- SVGs: `import icon from './x.svg?url'` for a URL; plain import gives an SVGR React component
- API calls go through `src/api/apiClient.ts` (retry + telemetry built in) — don't use fetch directly
```
Why: the second line states the target first and keeps the ban only beside it. A path is worth writing when the pointer *is* the trap; verify it exists first, since paths rot fastest.

### Environment and ordering constraints

```markdown
- `NEXT_PUBLIC_*` vars are baked at build time, not runtime
- Tests need `--runInBand` (shared DB state)
```

### Patterned script sets — compress, don't enumerate

```markdown
- Dev scripts follow `npm run dev:<app>` — `sf` = storefront, `adm` = admin panel, `mw` = middleware (abbreviations don't match folder names)
- `dev:*` scripts are non-terminating watchers — validate with `build`/`lint`/`test` instead of running them
```
Why: a 12-row patterned script list is still a package.json mirror. The residue is the naming pattern, the abbreviation legend (only when the abbreviations don't match folder names — that's a naming trap), and which scripts never terminate.

### Destructive workflow triggers and test asymmetries

```markdown
- Pushing to `main` with changes to `package.json`, `package-lock.json`, or `packages/app/**` triggers a production deploy (`.github/workflows/production-deploy.yml`)
- Only `packages/app` has tests — root `npm test` fans out with `--if-present`; the suite needs the framework-env mock in `src/__mocks__/` to run at all
```
Why: the deploy line quotes the workflow's exact branch and path filters, never a paraphrase. The test line names which workspaces have tests and what they need to run; the runner itself is one Read away.

### Repository etiquette

```markdown
- Branch names come from the ticket title; commit subjects are one line, no body
```
Why: every session that commits needs it, and no file path would load it on demand.

### Pointers to relocated content

```markdown
- Before writing or changing tests, read `docs/TESTING.md` (fixtures, the DB mock, naming)
```
Why: the wording decides whether the agent opens the doc. It names the task that should send the agent there and what it will find, with the path in backticks rather than as an `@` import.

## Diff format for proposals

```markdown
### Update: ./AGENTS.md

**Why:** <one line — which trap/error this prevents, or what falsehood/filler it removes>

```diff
- Next.js with Preact as React replacement (configured in next.config.ts)
+ Preact is in package.json but the aliasing in next.config.ts is commented out — the app runs on real React
```
```

For pure deletions, still state the why ("DERIVABLE: mirrors package.json scripts 1:1").

## Validation checklist

Before finalizing:

- [ ] Every kept or added claim verified against the file it's about, this session; every path exists; every `@` import resolves
- [ ] Nothing FILLER or DERIVABLE left in the root; directory maps deleted whole, not slimmed
- [ ] RELOCATABLE lines moved, not deleted; relocated docs consolidated — none so small (~<10 lines) it should merge into a sibling
- [ ] Every breadcrumb names the task that should open it and what's inside, as a plain path; spot-check one: would an agent following it find what it needs?
- [ ] Every ban sits beside its target; emphasis on one line at most
- [ ] No new line contradicts an existing one, in this file or one that loads beside it
- [ ] (Audit) every missing-trap item in Phase 3 looked at
- [ ] Root file got shorter, or longer only by trap/gotcha lines
- [ ] Every CLAUDE.md, at any level, starts with `@AGENTS.md` (or is a symlink to it) and holds only Claude-specific lines — no bare pointer unless a keep condition in loading.md holds, no duplicate
