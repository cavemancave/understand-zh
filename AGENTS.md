# AGENTS.md — Working rules for AI agents in this repo

This file records the project owner's standing instructions. Agents (Copilot CLI, etc.) MUST follow these rules when working in this repository.

## Identity / commits
- Use the project's original git identity: `cavemancave <echooffapple@gmail.com>`.
- This matches the existing history on `origin/main`. Set it per-worktree with `git config user.name cavemancave` and `git config user.email echooffapple@gmail.com` if the worktree's config differs.

## Commit & push policy
- One file per commit. After each successful change to a file, create a commit immediately.
- Commit messages are written in **English** (the codebase content is bilingual, but commit history stays English).
- Push after every commit (`git push`), so progress is visible upstream.
- Include the standard `Co-authored-by: Copilot ...` trailer when an AI agent assisted.

## Translation work
- Source: `original_html/understandNNN.html` (English).
- Target: `translated_html/understandNNN_zh.html` (Simplified Chinese).
- Preserve original HTML structure, anchors, links, code blocks, tables.
- Translate `alt` text on images.
- Keep the link to `understand_zh.css` intact.

## Figures
- Figures in the Chinese HTML files must be **inline `<svg>`** (not `<img>` to PNG).
- SVG text/labels are translated to Chinese.
- Convert one figure per commit to keep context small and review easy.
- Commit message format: `Inline SVG for figure N in understandNNN_zh (Chinese labels)`.

## Order of work — each unit in a fresh sub-agent / session
To avoid context attention deterioration, **every file translation and every figure-to-SVG conversion is performed by a fresh sub-agent (separate context window)**. The main session only orchestrates and verifies. A sub-agent receives full context (this AGENTS.md, the target file, the rules below) and is itself responsible for committing + pushing before exiting.

For each remaining English chapter file:
1. Sub-agent translates the file → commit → push.
2. For each figure in that chapter, a separate sub-agent converts it to inline SVG → commit → push.
3. Then move to the next chapter.

Retroactive PNG→SVG conversion for `understand005_zh..008_zh.html` follows the same one-sub-agent-per-figure rule.

## Progress tracking
- Persistent per-session plan lives in the agent's session workspace (`plan.md`) and the SQL `todos` table.
- This `AGENTS.md` is the long-lived rule book; update it when the owner's preferences change.
