# AGENTS.md Update Guidelines

## Core principle

AGENTS.md is loaded into every session. Every line costs context; a line earns its place in the root only if it is (a) verified against the code right now, (b) not derivable from an obvious file, (c) would change what an agent does, and (d) relevant to most sessions — the bar is **undiscoverable AND globally relevant**.

Content that passes (a)–(c) but is scoped to one domain gets **relocated**, not kept and not deleted: move it to a referenced doc (`docs/TESTING.md`) or a skill, leave a one-line breadcrumb. Phrase breadcrumbs conversationally — "For TS conventions, see docs/TYPESCRIPT.md" — no ALWAYS/NEVER, no all-caps forcing. Consolidate related topics into one doc rather than fragmenting — a handful of substantial docs beats many tiny ones (see the relocated-doc template and size rules in templates.md).

The default direction of an update is **removal or relocation**. Successful audits usually make the root file shorter.

## What TO add

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
Caveat: file paths are the fastest-rotting content in the file. A path reference is worth it when the pointer *is* the trap; keep such lines few and verify every path exists before writing it.

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
Why: a 12-row patterned script list is still a package.json mirror. The non-derivable residue is the naming pattern, the abbreviation legend (only when the abbreviations aren't obvious from folder names — that's a naming trap), and which scripts never terminate.

### Destructive workflow triggers and test asymmetries

```markdown
- Pushing to `main` with changes to `package.json`, `package-lock.json`, or `packages/app/**` triggers a production deploy (`.github/workflows/production-deploy.yml`)
- Only `packages/app` has tests — root `npm test` fans out with `--if-present`; the suite needs the framework-env mock in `src/__mocks__/` to run at all
```
Why: for outward-facing workflows (deploy, publish), quote the exact branch + path filters from the workflow file — a paraphrase on the scariest line is as bad as an unverified claim. For tests, the non-derivable part is which workspaces actually have them and what mocks/aliases they need to run; the runner and framework are one Read away and stay out.

## What NOT to add

### 1. Mirrors of package.json / README

Bad: `npm run dev — Start development server` (×10 rows). The agent reads `package.json` and treats it as the source of truth; the mirror only adds drift risk. Keep only the non-derivable residue (see above).

### 2. Tech-stack and architecture recitals

Bad: "Stack: Next.js, TypeScript, styled-components…" or a directory tree. Derivable in one Read/`ls`. Exception: a stack claim that is a *trap* — but then it must be verified (see below).

### 3. Unverified claims

Before writing "X is configured in Y", open Y and confirm. A dependency in `package.json` with its wiring commented out is *not* configured. A false line in AGENTS.md outranks the agent's own reading of the code — it is worse than saying nothing.

### 4. Aspirational / audience-mismatched content

Bad: "Protection against XSS, SQL injection, and OWASP Top 10; regular penetration testing." Not actionable, not verifiable, addressed to auditors. If enforcement exists, point at it: "form input sanitization: `src/helper/validate.ts`". If not, write nothing.

### 5. Generic advice and one-off history

"Write tests", "use meaningful names", "fixed login bug in abc123" — universal or non-recurring; cut.

### 6. Domain-scoped rule blocks (relocate, don't add)

Bad in the root: ten lines of TypeScript conventions, a testing-patterns section, API design rules. True and useful, but they tax every session that doesn't touch that domain. Put them in `docs/<DOMAIN>.md` or a skill and add one breadcrumb line. Also check for **contradictions** with existing lines before adding anything — conflicting instructions accumulated over time are worse than either instruction alone; surface the pair and let the user pick.

## Diff format for proposals

```markdown
### Update: ./AGENTS.md

**Why:** <one line — which trap/error this prevents, or what falsehood/filler it removes>

```diff
- Next.js with Preact as React replacement (configured in next.config.ts)
+ Preact is in package.json but the aliasing in next.config.ts is commented out — the app runs on real React
```
```

For pure deletions, still state the why ("mirrors package.json scripts 1:1; drift risk, no signal").

## Validation checklist

Before finalizing:

- [ ] Every kept/added claim verified against the file it's about, this session
- [ ] Every referenced path existence-checked
- [ ] No line derivable from one Read of package.json / README / `ls`
- [ ] No aspirational or audience-mismatched content
- [ ] No domain-scoped rule blocks in the root — relocated with breadcrumbs instead
- [ ] Relocated docs are consolidated — none so small (~<10 lines) it should merge into a sibling
- [ ] Spot-check one breadcrumb: would an agent following it actually find the guidance it needs?
- [ ] No new line contradicts an existing line
- [ ] Known traps in this repo are covered
- [ ] Root file got shorter, or longer only by trap/gotcha lines
- [ ] If a CLAUDE.md exists, it starts with `@AGENTS.md` and holds only Claude-specific lines — no bare pointer, no duplicate
