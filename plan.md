# Plan: Translate remaining chapters + convert figures to inline SVG

## User's standing instructions ("order")
1. Maintain a doc to remember the user's order (this plan.md in session + AGENTS.md in repo).
2. Use the project's original git identity `cavemancave <echooffapple@gmail.com>` (matches origin/main history). Override the current worktree's `user.name`/`user.email` to this if needed.
3. Translate the remaining English HTML files into Chinese.
4. Commit messages are written in English.
5. Commit each file after a successful change (one file = one commit). Push after every commit.
6. Replace figures in the Chinese HTML files with inline `<svg>`. SVG text/labels are translated to Chinese. Each figure is a separate task/commit to avoid context overload.

## Current state
- Translated: `understand001_zh.html` .. `understand009_zh.html`.
- Remaining English source files to translate: `understand010.html` .. `understand029.html` (20 files: chapters 7–14 + appendices A–K).
- Figures in `original_html/figures/`: 81 PNGs (`understand-html001.png` .. `understand-html081.png`).
- Figures already referenced in existing zh files: 28 (in `understand005_zh.html`..`understand008_zh.html`).

## Workflow (per-chapter, then per-figure) — each unit runs in a fresh sub-agent

To avoid attention/context deterioration, every translation file and every figure-to-SVG conversion is delegated to a **fresh sub-agent (separate context window)** via the `task` tool. The main session only orchestrates: picks the next todo, launches a sub-agent with complete context, then commits+pushes (or has the sub-agent commit+push) and moves on.

For each remaining English source file `understandNNN.html`:
1. **Launch sub-agent A (translation)** — prompt includes: source path, target path, style rules from `AGENTS.md`, css link convention, leave image refs unchanged for now, translate `alt` text. Sub-agent writes the file, runs sanity checks, then `git add` + `git commit` (English message) + `git push`.
2. Main session verifies commit landed.
3. For each figure referenced by that chapter, **launch sub-agent B (one figure)** — prompt includes: PNG path, target zh HTML, the exact `<img>` tag to replace, instructions to author inline `<svg>` with Chinese labels, then commit+push.
4. Move to next chapter.

Retroactive SVG conversion for `understand005_zh..008_zh.html` (28 figures) follows the same one-sub-agent-per-figure rule.

## Sub-agent contract
- Type: `general-purpose` (full toolset, Sonnet) for translation; `general-purpose` for SVG too (geometry + commit work).
- Each sub-agent is stateless: the launching prompt MUST contain all rules (git identity, commit-per-file, English message, push after commit, link to `AGENTS.md`).
- Sub-agent is responsible for its own commit + push so the main context stays small.
- Main session updates the SQL `todos` row to `done` once the sub-agent reports success and the commit/push is confirmed.

## Commit message conventions (English)
- Translation: `Translate understandNNN to Chinese (chapter X / appendix Y)`
- SVG: `Inline SVG for figure N in understandNNN_zh (Chinese labels)`

## Todos
Tracked in the SQL `todos` table. Two kinds of todos:
- `xlate-NNN` — translate `understandNNN.html`
- `svg-NNN-K` — convert figure K used in `understandNNN_zh.html` to inline SVG

## Notes / considerations
- Before first commit, set `git config user.name cavemancave` and `git config user.email echooffapple@gmail.com` in this worktree.
- Keep `understand_zh.css` link tag unchanged in each translated file.
- For SVG: prefer hand-authored geometric SVG that approximates the original diagram; do not embed the PNG as base64.
- If a figure is complex enough that recreation risks errors, flag it as `blocked` in todos and ask the user.
