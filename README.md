# agents-md-audit

Keep agent context files **slim, verified, and worth their context cost**.

A personalized rebuild of Anthropic's official [claude-md-management](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management) plugin. The original audited CLAUDE.md files against a points rubric that rewarded comprehensiveness (commands documented, architecture described) — which in practice produced bloated files full of `package.json` mirrors and unverified claims. This rebuild inverts that.

## Philosophy

- **AGENTS.md is the canonical context file.** CLAUDE.md is a thin pointer:

  ```markdown
  # CLAUDE.md

  @AGENTS.md

  ## Claude Code

  (anything Claude-specific; omit if empty)
  ```

  The skill creates the pointer when it's missing and migrates full CLAUDE.md files into this layout.

- **Every claim is verified against the code**, not against `package.json` or an old README. A false claim (e.g. "uses Preact" when the aliasing is commented out) is worse than no claim.
- **Derivable content is cut**: no mirrors of `package.json` scripts, no tech-stack recitals, no directory maps. Only the non-derivable residue stays — what an umbrella script chains, what CI gates on, naming traps, gotchas.
- **Domain-scoped rules are relocated**, not kept or deleted: TypeScript conventions, testing patterns etc. move to `docs/<DOMAIN>.md` with a one-line breadcrumb (progressive disclosure). The root bar is *undiscoverable AND globally relevant*.
- **No aspirational filler**: security/quality posture statements are deleted unless they point to actual enforcement code.
- **Near-empty is a valid outcome.** If a repo has no traps, the correct AGENTS.md is a few lines — never padded to look complete.

## Why it works this way

Anthropic's [new rules of context engineering for Claude 5 generation models](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models) describes the same failure modes this skill exists to remove:

| Their guidance | What the skill does |
|---|---|
| "Keep your CLAUDE.md lightweight … spend most of the tokens on gotchas inside of the codebase." | Keeps the traps and gotchas. Cuts the rest. |
| "Avoid stating 'the obvious' things Claude should know by looking at your file system or your repo." | Cuts derivable content — `package.json` mirrors, tech-stack recitals, directory maps. |
| "Use progressive disclosure heavily … create a verification skill and reference it from your CLAUDE.md." | Moves domain-scoped rules into `docs/<DOMAIN>.md` or a skill, leaving a one-line breadcrumb. |
| Then: give Claude rules → **Now: let Claude use judgement.** | Deletes over-constraining rules and posture statements no code enforces. |
| Then: memory in CLAUDE.md → **Now: auto-memory.** | Treats the file as durable project truth, not a scratchpad for session notes. |

The short version: a context file earns its tokens by holding what an agent **cannot** infer from the repo itself. Everything else is a tax paid on every single session.

## What's inside

| | agents-md-audit (skill) | /revise-agents-md (command) |
|---|---|---|
| **Purpose** | Audit AGENTS.md against the codebase, fix false claims, cut filler, relocate scoped rules | Capture verified session learnings into AGENTS.md |
| **Trigger** | "audit my AGENTS.md / CLAUDE.md" | End of session |
| **Typical outcome** | A shorter, truer file (plus the pointer CLAUDE.md) | A few trap/gotcha lines added, stale lines removed |

The skill classifies every line as false / aspirational / derivable / relocatable / earned, reports before editing, and always states a layout action. See `skills/agents-md-audit/references/` for the quality criteria, templates, and update guidelines.

## Why not just `/doctor`?

Claude Code's built-in [`/doctor`](https://code.claude.com/docs/en/commands) overlaps a lot, and it's worth being upfront about that: it trims checked-in CLAUDE.md files by cutting content Claude could derive from the codebase, migrates what remains into skills and nested files that load on demand, and reports before changing anything. If that's all you need, use it — it ships in the box.

This skill is narrower, and differs in three ways:

- **AGENTS.md is the target.** `/doctor` keeps guidance in CLAUDE.md and nested CLAUDE.md files. This skill makes AGENTS.md canonical and CLAUDE.md a one-line pointer, so Cursor, Codex, and anything else reading the [open standard](https://agentskills.io) sees the same context.
- **It asks whether claims are *true*, not just whether they're needed.** Derivability and accuracy are different axes. `/doctor`'s trim is about context cost; this skill re-opens the code behind every surviving line, and a false claim is the highest-priority fix.
- **It runs outside Claude Code.** Installed through the `skills` CLI, it works in any agent that supports skills. `/doctor` is Claude Code only.

`/doctor` also has no equivalent of `/revise-agents-md` — folding what you learned in a session back into the file.

## Install

As a skill — works with Claude Code, Cursor, and any agent the [`skills`](https://www.skills.sh) CLI supports:

```sh
npx skills add vr1e/claude-md-audit
```

Listed at [skills.sh/vr1e/claude-md-audit](https://skills.sh/vr1e/claude-md-audit/agents-md-audit).

As a Claude Code plugin — also installs the `/revise-agents-md` command:

```
/plugin marketplace add vr1e/claude-md-audit
/plugin install agents-md-audit@claude-md-audit
```

## Usage

Ask for an audit in plain language — the skill triggers on its own:

> audit my AGENTS.md

> is anything in CLAUDE.md actually wrong?

At the end of a session, capture what you learned (plugin install only):

```
/revise-agents-md
```

## Layout

```text
skills/agents-md-audit/
├── SKILL.md                        # the audit skill
└── references/
    ├── quality-criteria.md         # the false / derivable / relocatable / earned classification
    ├── templates.md                # slim AGENTS.md, CLAUDE.md pointer, relocated-doc templates
    └── update-guidelines.md        # add/don't-add criteria for session learnings
commands/
└── revise-agents-md.md             # /revise-agents-md
```

## License

MIT — see [LICENSE](LICENSE).
