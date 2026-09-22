# AGENTS.md Quality Criteria

Do not use a points rubric. Rubrics that award points for "commands documented" and "architecture clarity" reward exactly the bloat this skill exists to remove. Instead, classify every line of the file into one of five buckets and judge the file by its bucket ratio.

## The five buckets

### 1. FALSE — highest severity, fix first

The claim contradicts what the code actually does. Typical cause: written from `package.json` or an old README without opening the referenced file.

Canonical example: "Next.js with Preact as React replacement (configured in next.config.ts)" — while in `next.config.ts` the aliasing is commented out. Preact being in `package.json` doesn't make the claim true. An agent trusting it reasons about preact/compat quirks that don't exist.

**Rule: a claim is only verified by the file it is about, at audit time.** Dependencies present ≠ used. Config file exists ≠ config active.

### 2. UNVERIFIABLE / ASPIRATIONAL — delete

Statements about process, intent, or posture that no file in the repo can confirm: "regular penetration testing", "protection against OWASP Top 10", "we follow clean-code principles". Addressed to auditors, not agents. If a real enforced mechanism exists, replace with its location: "input sanitization: `src/helper/sanitize.ts`, applied in form handlers".

### 3. DERIVABLE — cut, keep only the non-derivable residue

The agent could get this in one cheap Read of an obvious source-of-truth file:

- Command lists mirroring `package.json` scripts ("`npm run dev` — start dev server")
- Tech-stack enumerations
- Directory trees restating what `ls` shows
- Anything duplicated from README

These cost context every session, add no signal, and drift. The residue worth keeping is what the source-of-truth *can't* tell you:

- What an umbrella script chains ("`npm run lint` = tsc → eslint → stylelint")
- Which command CI gates on
- Required env vars, ordering constraints, "this isn't an npm script"
- For large patterned script sets: the pattern + abbreviation legend (one line, not the 12-row list) and which scripts are non-terminating watchers an agent shouldn't run

### 4. RELOCATABLE — move out of the root, leave a breadcrumb

True, verified, and non-derivable — but scoped to one domain: TypeScript conventions, testing patterns, API design rules, git workflow. The root file loads on every session; a React styling rule taxes every backend and docs session it's irrelevant to. Move it to a referenced doc (`docs/TESTING.md`) or a skill, and leave a one-line conversational breadcrumb: "For TS conventions, see docs/TYPESCRIPT.md". Referenced docs can reference deeper docs — agents navigate hierarchies well.

Deleting relocatable content is as much a failure as keeping it in the root.

### 5. EARNED — keep in the root

Verified, non-derivable, **globally relevant**, and behavior-changing: an agent without this line would do something wrong or waste real time, in most sessions.

- Naming traps: `pages/` (routes) vs `src/pages/` (view components); root entry is `proxy.ts`, formerly `middleware.ts`
- Non-obvious conventions: SVG `?url` suffix vs SVGR component import
- Concrete pointers with a why: retry logic in `apiClient.ts`, the layout HOC every page must wrap itself in
- Gotchas that caused or would cause a debugging session
- Environment facts the agent cannot discover ("you are on WSL — path resolution differs")

Note on paths: EARNED lines may reference files (`apiClient.ts`, `proxy.ts`) when the pointer *is* the trap — but paths are the fastest-rotting content in the file. Keep them few, and every path must pass an existence check at audit time.

## Verdict

- **Good**: mostly EARNED, zero FALSE, ≲60 lines. Report "keep, minor trims".
- **Needs work**: any FALSE claim, any unresolved contradiction, or >⅓ DERIVABLE/ASPIRATIONAL/RELOCATABLE. Propose targeted edits.
- **Rebuild**: template boilerplate never customized, or more filler than signal. Propose a fresh slim AGENTS.md from actual codebase findings.

Also always check for **missing EARNED content**: scan the repo for confusable names, umbrella scripts, and unusual entry points the file doesn't mention. An accurate-but-silent file still fails an agent walking into a trap.

## Red flags checklist

- Claim about a config file you haven't opened this audit
- "Configured in X" where X shows it commented out or unused
- 1:1 script mirror of `package.json`
- Security/quality posture paragraphs
- Referenced paths that don't exist
- Contradicting instructions (usually added by different developers over time) — surface the pair, make the user pick
- Domain-scoped rule blocks in the root (TS conventions, testing patterns) — relocate, don't keep or delete
- Same content in both CLAUDE.md and AGENTS.md — AGENTS.md is canonical; CLAUDE.md holds only `@AGENTS.md` plus Claude-specific lines, or doesn't exist
- A CLAUDE.md next to an AGENTS.md that doesn't start with `@AGENTS.md` — Claude Code is silently ignoring AGENTS.md
- File longer than ~80 lines — almost always contains derivable filler or relocatable content
