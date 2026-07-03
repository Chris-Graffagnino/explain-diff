---
name: explain-diff
description: Explains a code diff, commit, branch, or PR at three depths — summary (quick triage for humans or AI agents), teach-me (teaching walkthrough for novice coders), or expert (dense expert analysis with a review-focus map highlighting likely bugs, regressions, and design problems). Use when the user asks to "explain this diff", "explain this PR", "what does this change do", "walk me through these changes", or wants to understand a code change before reviewing or merging it. Supports --html for a rich interactive page and "help" for usage. Do NOT use for verified bug-hunting or posting review verdicts — hand off to a dedicated code-review pass for that (e.g. /code-review in Claude Code).
metadata:
  author: chrisgraffagnino
  version: 1.0.0
---

# Explain Diff

Build a genuine understanding of a code change, then explain it at the depth the reader needs. The explanation serves two audiences at once: a human trying to understand what changed and why, and a reviewer (human or AI) deciding where to focus attention to catch bugs, regressions, and design problems.

This skill **explains and directs attention**. It does not produce verified bug findings — when a risk callout deserves confirmation, recommend a dedicated code-review pass as the follow-up (`/code-review` in Claude Code).

## Step 0: Help

If the arguments are exactly `help`, `--help`, `-h`, or `?`, print the help block below verbatim and stop. Do not read files, do not gather a diff.

```
explain-diff — explain a code change for humans or AI

Usage: /explain-diff [mode] [--html] [target]

Modes:
  summary   (default) Quick orientation: what changed, why, what behavior
            moved, and where the risk concentrates. ~1 screen of output.
  teach-me  Teaching walkthrough for developers new to the codebase or
            domain: background on the surrounding system, intuition with
            toy examples, a guided code tour, and a 5-question quiz.
  expert    Dense expert analysis: intent and approach, invariants added
            and removed, cross-file impact, and a ranked review-focus map
            for hunting bugs, regressions, and design problems.

Flags:
  --html    Also write a self-contained interactive HTML page (table of
            contents, diagrams, clickable quiz in teach-me mode) to
            /tmp/YYYY-MM-DD-explanation-<slug>.html

Target (optional, defaults to your working diff):
  <empty>        Branch diff vs upstream/main, plus uncommitted changes
  <PR number>    e.g. 4821 — fetched via `gh pr diff`
  <git range>    e.g. main..HEAD, abc123..def456
  <commit>       e.g. HEAD~1, or a commit SHA
  <branch>       compared against the repo's default branch
  <path>         limit the explanation to a file or directory

Examples:
  /explain-diff                     summary of your current working diff
  /explain-diff teach-me 4821       teaching walkthrough of PR #4821
  /explain-diff expert main..HEAD   expert analysis of the branch
  /explain-diff summary src/auth/   what changed under src/auth/

Related: a dedicated code-review command (e.g. /code-review in Claude
Code) finds and verifies bugs; explain-diff builds understanding and
points at where the risk concentrates.
```

## Step 1: Parse arguments

- **Mode**: if the first token is `summary`, `teach-me`, or `expert`, that is the mode. Otherwise default to `summary`.
- **Flags**: `--html` may appear anywhere.
- **Target**: whatever remains — a PR number, git range, commit, branch, or path. May be empty.

If a token is ambiguous (e.g. a branch named `expert`), prefer the mode reading only for the first token and say which interpretation you chose.

## Step 2: Acquire the diff

- **No target**: run `git diff @{upstream}...HEAD` (fall back to `git diff main...HEAD`, then `git diff HEAD~1` if there's no upstream). If there are uncommitted changes, or the range diff is empty, also run `git diff HEAD` and include the working-tree changes in scope.
- **PR number**: `gh pr view <n>` for the title/description, `gh pr diff <n>` for the diff.
- **Git range / commit / branch**: `git diff <range>`; for a branch, diff against the repo's default branch. Read the commit messages in the range too (`git log`) — they carry stated intent.
- **Path**: restrict the diff commands above to that path.

If the diff is empty, say so and stop — do not invent an explanation.

## Step 3: Build understanding (all modes, before writing anything)

Do the investigation first; the writing is only as good as this step.

1. **Read every hunk, then the full enclosing function or class** for each hunk. Behavior often lives in the unchanged lines around a change.
2. **Read stated intent**: commit messages, PR title and description, linked issues if cheap to fetch. Keep "stated intent" separate from what you infer.
3. **Explore the surrounding system**: the modules the change touches, callers of changed symbols (Grep), the tests that cover this area, and any config or schema the change depends on. For teach-me mode, go broader — you'll need to explain the neighborhood, not just the change.
4. **Run the analysis lenses** in `references/analysis-lenses.md`: change taxonomy, removed-behavior audit, cross-file impact, invariant delta, design pressure, and test delta. These produce the raw material for the risk callouts in every mode.
5. **Group the hunks into themes** — a handful of narrative units ordered by importance, not by file path. A theme is something like "switch the cache key to include the tenant id," not "changes to cache.py."

Scale the depth to the diff: a 10-line change needs minutes of this, not an expedition. For very large diffs (roughly 2,000+ changed lines), work theme-by-theme, say explicitly that you are summarizing at theme level, and offer to zoom into any theme on request.

## Step 4: Write the explanation for the chosen mode

Read `references/output-formats.md` before writing — it owns the full output contract for every mode: section structure, skeletons, budgets, caps, and voice. The gist of each mode:

- **summary** (default) — one-screen orientation a maintainer or an AI agent can absorb in under a minute: title, what/why, a change map by theme, user-/API-/data-visible behavior changes, and a short ranked "watch out for" list. If the change is trivial, two sentences and "Nothing notable to watch for" is the correct output — do not pad.
- **teach-me** — a teaching document for a developer new to the codebase or the domain: background, the core idea walked through with a running toy example, a guided code tour, what could go wrong (taught as review thinking), and a quiz. Write with the clarity and flow of Martin Kleppmann: concrete before abstract, smooth transitions, no condescension.
- **expert** — dense analysis for a domain expert, zero background, every sentence load-bearing: intent & approach, delta semantics (including the removed-behavior audit), cross-file impact, a ranked review-focus map — the payload that makes this mode useful for PR review — and open questions for the author.

## Step 5: HTML output (only if `--html` was passed)

Produce the markdown explanation in the terminal as usual, **and** write a single self-contained HTML file (inline CSS and JS) following the spec in `references/output-formats.md` § HTML output. Filename: `/tmp/YYYY-MM-DD-explanation-<slug>.html` with today's actual date, so files stay time-sorted and out of version control. Tell the user the path when done.

## Honesty rules

- **Never present a suspicion as a confirmed bug.** Risk callouts are attention pointers. When one looks serious, say so and recommend a dedicated code-review pass to verify it (`/code-review` in Claude Code).
- **Distinguish stated intent from inferred intent.** "The PR description says…" vs "This appears to be…".
- **Don't invent content to fill sections.** Empty "Behavior changes" or "Watch out for" sections should say "None" — a reader who learns to trust that will move faster.
- **Quote real code.** Every claim about what the code does should survive the reader opening the file at the cited line.

## Examples

**Example 1** — User says: "explain this PR" (on a branch with an open PR)
Actions: resolve the PR via `gh pr view`, gather diff and description, investigate, run lenses, write **summary** output.
Result: one-screen orientation with change map, behavior changes, and ranked watch-outs.

**Example 2** — User says: "/explain-diff teach-me 4821 --html"
Actions: fetch PR #4821, investigate broadly (teach-me needs neighborhood context), write the teaching walkthrough in the terminal, then generate `/tmp/2026-07-02-explanation-tenant-cache-keys.html` with TOC, diagrams, and a clickable quiz.
Result: markdown walkthrough plus the HTML file path.

**Example 3** — User says: "/explain-diff expert before I review this"
Actions: gather the working diff, run all lenses with emphasis on removed-behavior and cross-file impact, write **expert** output.
Result: delta semantics plus a ranked review-focus map; ends by offering a dedicated code-review pass to verify the serious callouts.

## Troubleshooting

**Empty diff** — Cause: clean working tree and no commits ahead of upstream. Solution: report exactly what you compared (e.g. "no diff between HEAD and origin/main, working tree clean") and ask what the user wants explained.

**`gh` fails for a PR target** — Cause: not authenticated or no remote. Solution: say so, suggest running `gh auth login`, and offer to explain a local range instead.

**Diff too large to explain line-by-line** — Cause: 2,000+ changed lines or generated/vendored files. Solution: exclude lockfiles and generated code from the narrative (note that you did), explain at theme level, and offer to zoom into any theme.

**Mode token collides with a branch name** — Solution: prefer the mode reading, state the interpretation, and mention the user can force the other reading with an explicit range (e.g. `main..expert`).
