# explain-diff

A [`SKILL.md`](https://agentskills.io) agent skill that explains a code diff, commit, branch, or PR at the depth the reader needs — from a one-screen triage summary to a teaching walkthrough to dense expert analysis with a review-focus map.

It runs in any tool that reads the open `SKILL.md` skill standard — including [Claude Code](https://claude.com/claude-code) and [OpenAI Codex CLI](https://developers.openai.com/codex/skills) — from a single shared skill definition.

It serves two audiences at once: a human trying to understand *what changed and why*, and a reviewer (human or AI) deciding *where to focus attention* to catch bugs, regressions, and design problems.

> **explain-diff explains and directs attention. It does not produce verified bug findings.** When a risk callout deserves confirmation, it recommends a dedicated code-review pass (`/code-review` in Claude Code) as the follow-up.

## Modes

| Mode | Audience | Output |
| --- | --- | --- |
| `summary` *(default)* | A maintainer triaging a review queue, or an AI agent loading context | ~1 screen: title, what/why, a change map by theme, behavior changes, and a ranked "watch out for" list |
| `teach-me` | A developer new to the codebase or domain | Background, the core idea walked through a running toy example, a guided code tour, review-thinking, and a 5-question quiz |
| `expert` | A domain expert who needs zero background | Intent & approach, delta semantics, cross-file impact, and a ranked review-focus map with concrete failure scenarios |

## Usage

```
/explain-diff [mode] [--html] [target]
```

**Target** (optional, defaults to your working diff):

- *empty* — branch diff vs upstream/main, plus uncommitted changes
- a PR number — e.g. `4821`, fetched via `gh pr diff`
- a git range — e.g. `main..HEAD`, `abc123..def456`
- a commit — e.g. `HEAD~1` or a SHA
- a branch — compared against the repo's default branch
- a path — limit the explanation to a file or directory

**Flags:**

- `--html` — also write a self-contained interactive HTML page (table of contents, diagrams, clickable quiz in teach-me mode) to `/tmp/YYYY-MM-DD-explanation-<slug>.html`

### Examples

```
/explain-diff                     summary of your current working diff
/explain-diff teach-me 4821       teaching walkthrough of PR #4821
/explain-diff expert main..HEAD   expert analysis of the branch
/explain-diff summary src/auth/   what changed under src/auth/
/explain-diff help                usage
```

## How it works

Before writing anything, the skill builds genuine understanding of the change: it reads every hunk *and* the enclosing function, separates stated intent (commit messages, PR description) from inferred intent, explores callers and tests around the change, and runs a set of **analysis lenses** — change taxonomy, removed-behavior audit, cross-file impact, invariant/contract delta, design-pressure check, test delta, and hidden-risk heuristics. Those lenses produce the change map and the ranked risk callouts that appear in every mode.

It follows a few honesty rules throughout: never present a suspicion as a confirmed bug, distinguish stated from inferred intent, never invent content to fill a section (empty sections say "None"), and quote real code so every claim survives the reader opening the file.

## Installation

The repository root *is* the skill directory (it contains `SKILL.md`). Both tools load a skill by discovering a directory named after the skill, so install by symlinking (or copying) this repo into the right skills folder under the name `explain-diff`.

```sh
git clone https://github.com/Chris-Graffagnino/explain-diff.git
```

**Claude Code** — reads `~/.claude/skills/` (user) or `.claude/skills/` (project):

```sh
ln -s "$(pwd)/explain-diff" ~/.claude/skills/explain-diff        # every project
# — or, project-only:
mkdir -p .claude/skills && ln -s "$(pwd)/explain-diff" .claude/skills/explain-diff
```

**OpenAI Codex CLI** — reads `~/.agents/skills/` (user) or `.agents/skills/` (project, scanned up to the repo root):

```sh
ln -s "$(pwd)/explain-diff" ~/.agents/skills/explain-diff        # every project
# — or, project-only:
mkdir -p .agents/skills && ln -s "$(pwd)/explain-diff" .agents/skills/explain-diff
```

Both tools trigger the skill implicitly when your request matches its description ("explain this PR"), and you can invoke it explicitly — `/explain-diff` in Claude Code, or via the `/skills` menu / `$explain-diff` in Codex. PR targets require the [`gh` CLI](https://cli.github.com/) to be installed and authenticated.

> Codex reads the same `SKILL.md`. The `agents/openai.yaml` file adds Codex-only UI polish (display name, brand color) and is harmlessly ignored by Claude Code.

## Repository layout

```
explain-diff/                       # this repo — the skill directory itself
├── SKILL.md                        # skill entry point and step-by-step procedure (shared)
├── AGENTS.md                       # project instructions pointing at the skill (Codex)
├── references/
│   ├── analysis-lenses.md          # the 7 lenses run while building understanding
│   └── output-formats.md           # per-mode output contract + HTML spec
└── agents/
    └── openai.yaml                 # optional Codex-only metadata (ignored elsewhere)
```

## Related

A dedicated code-review pass (e.g. `/code-review` in Claude Code) finds and *verifies* bugs and posts review verdicts. explain-diff builds understanding and points at where the risk concentrates — reach for it *before* a review, then run a verification pass to confirm the serious callouts.

## Credits

Inspired by the original `explain-diff` skill by Geoffrey Litt ([@geoffreylitt](https://github.com/geoffreylitt)).

## License

[MIT](./LICENSE) © Chris Graffagnino
