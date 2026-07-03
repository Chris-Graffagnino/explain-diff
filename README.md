# explain-diff

A [Claude Code](https://claude.com/claude-code) skill that explains a code diff, commit, branch, or PR at the depth the reader needs — from a one-screen triage summary to a teaching walkthrough to dense expert analysis with a review-focus map.

It serves two audiences at once: a human trying to understand *what changed and why*, and a reviewer (human or AI) deciding *where to focus attention* to catch bugs, regressions, and design problems.

> **explain-diff explains and directs attention. It does not produce verified bug findings.** When a risk callout deserves confirmation, it recommends `/code-review` as the follow-up.

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

Clone the repo and make the skill discoverable to Claude Code by symlinking (or copying) the inner `explain-diff/` skill directory into your skills folder:

```sh
git clone https://github.com/Chris-Graffagnino/explain-diff.git

# User-level (available in every project):
ln -s "$(pwd)/explain-diff/explain-diff" ~/.claude/skills/explain-diff

# — or project-level (this repo only):
mkdir -p .claude/skills
ln -s "$(pwd)/explain-diff/explain-diff" .claude/skills/explain-diff
```

Then invoke it with `/explain-diff` inside Claude Code. PR targets require the [`gh` CLI](https://cli.github.com/) to be installed and authenticated.

## Repository layout

```
explain-diff/
├── SKILL.md                        # skill entry point and step-by-step procedure
└── references/
    ├── analysis-lenses.md          # the 7 lenses run while building understanding
    └── output-formats.md           # per-mode output contract + HTML spec
```

## Related

`/code-review` finds and *verifies* bugs and posts review verdicts. explain-diff builds understanding and points at where the risk concentrates — reach for it *before* a review, then run `/code-review` to confirm the serious callouts.

## Credits

Inspired by the original `explain-diff` skill by Geoffrey Litt ([@geoffreylitt](https://github.com/geoffreylitt)).

## License

[MIT](./LICENSE) © Chris Graffagnino
