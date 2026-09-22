---
description: Update AGENTS.md with learnings from this session
allowed-tools: Read, Edit, Write, Glob
---

Review this session for learnings that would help future agent sessions in this codebase, and fold them into **AGENTS.md** (the canonical context file; Claude Code reads it directly, so a CLAUDE.md exists only for Claude-specific guidance and must start with `@AGENTS.md`).

## Step 1: Reflect

What was missing from AGENTS.md that cost time this session?
- A trap you fell into (misleading name, wrong assumption the file could have prevented)
- A non-obvious command behavior (what an umbrella script chains, a required env var, what CI gates on)
- A convention you had to discover by reading code (and would need again)
- A claim in AGENTS.md that turned out to be **wrong** — correcting these is the highest priority

Explicitly skip: anything derivable from one Read of `package.json`/`README.md`, generic advice, one-off fixes.

For the full add/don't-add criteria, diff format, and validation checklist, read `${CLAUDE_PLUGIN_ROOT}/skills/agents-md-audit/references/update-guidelines.md`.

Root AGENTS.md only gets learnings relevant to *most* sessions. A domain-scoped learning (a testing pattern, a TS convention) goes into the referenced doc for that domain (`docs/<DOMAIN>.md`, creating it with a breadcrumb in AGENTS.md if needed) — not into the root. Check the new line doesn't contradict an existing one.

## Step 2: Find the files

Use Glob to locate the context files: patterns `AGENTS.md`, `CLAUDE.md`, `CLAUDE.local.md`, plus `*/AGENTS.md` and `*/CLAUDE.md` for nested ones (skip `node_modules`).

- Learnings go to `AGENTS.md`. If only a full CLAUDE.md exists, offer to migrate it: content → AGENTS.md; delete CLAUDE.md, or keep just `@AGENTS.md` + any Claude-specific lines.
- Personal/local-only notes go to `CLAUDE.local.md` (gitignored). If the repo has no CLAUDE.md, start it with `@AGENTS.md` — a CLAUDE.local.md on its own makes Claude Code skip AGENTS.md.
- Claude-specific items (skills, hooks, permissions) go under `## Claude Code` in CLAUDE.md, creating it with `@AGENTS.md` as its first line if needed.

## Step 3: Draft, then verify

One line per learning. **Before proposing any factual claim, re-open the file it's about and confirm it** — never write "X is configured in Y" from memory. A wrong line in AGENTS.md is worse than no line.

## Step 4: Show proposed changes

```
### Update: ./AGENTS.md

**Why:** [one line — what error or wasted time this prevents]

```diff
+ [the addition — brief, verified]
- [any wrong/stale line being removed]
```
```

Prefer diffs that also *remove* something. If the session revealed a line was derivable filler, propose cutting it.

## Step 5: Apply with approval

Ask before editing. Only touch files the user approves.
