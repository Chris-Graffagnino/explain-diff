# AGENTS.md

This repository **is** an agent skill: `explain-diff`. The skill definition
lives in [`SKILL.md`](./SKILL.md) with supporting material in
[`references/`](./references). It follows the open [`SKILL.md`](https://agentskills.io)
standard and runs in any compatible tool (OpenAI Codex CLI, Claude Code, etc.).

## Using the skill

When a request is about understanding a code change — "explain this diff",
"explain this PR", "what does this change do", "walk me through these changes",
or building understanding of a change before reviewing or merging it — load and
follow [`SKILL.md`](./SKILL.md). It is the source of truth for the procedure,
the three modes (`summary` / `teach-me` / `expert`), argument parsing, and the
output contract.

Do **not** use this skill to produce verified bug findings or post review
verdicts — it directs attention, it does not confirm bugs. Hand serious risk
callouts off to a dedicated code-review pass.

## Working on this repository

- `SKILL.md` and `references/` are the shared, tool-neutral skill content —
  keep them free of hard dependencies on any single agent tool.
- `agents/openai.yaml` is optional Codex-only metadata; other readers ignore it.
- Local, machine-specific settings under `.claude/` are gitignored — do not
  commit them.
