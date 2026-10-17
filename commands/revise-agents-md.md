---
description: Update AGENTS.md with learnings from this session
allowed-tools: Read, Edit, Write, Glob
disable-model-invocation: true
---

Review this session for learnings that would help future agent sessions in this codebase, and fold them into **AGENTS.md** (the canonical context file; Claude Code reads it directly, so a CLAUDE.md exists only for Claude-specific guidance and must start with `@AGENTS.md`).

## Step 1: Reflect

What was missing from AGENTS.md that cost time this session?
- A correction the user typed — the strongest signal: what they had to tell you this session, the next one will need too
- A trap you fell into (misleading name, wrong assumption the file could have prevented)
- A non-obvious command behavior (what an umbrella script chains, a required env var, what CI gates on)
- A convention you had to discover by reading code (and would need again)
- A claim in AGENTS.md that turned out to be **wrong** — correcting these is the highest priority

Sort each learning with the five buckets in Phase 3 of `${CLAUDE_PLUGIN_ROOT}/skills/agents-md-audit/SKILL.md` (read Phase 3 only; the rest is the full audit). Only an EARNED learning gets a root line. A RELOCATABLE one goes to its destination: the directory's AGENTS.md, or `docs/<DOMAIN>.md` with a breadcrumb in AGENTS.md that passes the Phase 3 pointer check. Check the new line doesn't contradict an existing one.

For examples of EARNED lines, the diff format, and the validation checklist, read `${CLAUDE_PLUGIN_ROOT}/skills/agents-md-audit/references/update-guidelines.md`.

## Step 2: Find the files

Use Glob to locate the context files at any depth: patterns `**/AGENTS.md`, `**/CLAUDE.md`, `**/CLAUDE.local.md` (skip `node_modules`).

- Learnings go to `AGENTS.md`. If only a full CLAUDE.md exists, offer to migrate it: content → AGENTS.md; delete CLAUDE.md, or keep just `@AGENTS.md` + any Claude-specific lines.
- Personal notes go to the user's `~/.claude/CLAUDE.md` or auto memory, which load alongside AGENTS.md. A `CLAUDE.local.md` (gitignored, this repo only) also works, but it turns off Claude Code's reading of AGENTS.md: start it with `@AGENTS.md` unless a CLAUDE.md already imports it.
- Claude-specific items (skills, hooks, permissions) go under `## Claude Code` in CLAUDE.md, creating it with `@AGENTS.md` as its first line if needed.

## Step 3: Draft, then verify

One line per learning. **Before proposing any factual claim, re-open the file it's about and confirm it** — never write "X is configured in Y" from memory. A wrong line in AGENTS.md is worse than no line.

## Step 4: Show proposed changes

Use the diff format in update-guidelines.md, one **Why** line per change. Prefer diffs that also *remove* something: if the session showed a line was FILLER or DERIVABLE, propose cutting it.

## Step 5: Apply with approval

Ask before editing. Only touch files the user approves.
