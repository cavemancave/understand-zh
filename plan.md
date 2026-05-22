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

## Workflow (UPDATED 2026-05-21: translate ALL chapters first, then convert figures)

User's revised preference: finish translating every remaining English chapter into Chinese FIRST (PNG `<img>` refs kept as-is), and only AFTER all chapters are translated, go back and convert figures to inline SVG, chapter by chapter.

Each unit (one file translation, one figure conversion) still runs in a **fresh sub-agent (separate context window)** via the `task` tool. The main session only orchestrates.

### Phase 1 — Translate all remaining chapters
For each remaining English source file `understandNNN.html` (013 → 029):
1. Launch a fresh sub-agent A (translation): source path, target path, style rules, css link convention, leave image refs as PNGs, translate `alt` text. Sub-agent writes the file, then `git add` + `git commit` (English message) + `git push`.
2. Verify commit landed, then move to next chapter.

### Phase 2 — Convert PNG figures to inline SVG
After Phase 1 is complete, walk chapters in order. For each PNG `<img>` in each `understandNNN_zh.html`, launch a fresh sub-agent B that inspects the source PNG, authors a hand-drawn inline `<svg>` with Chinese labels, replaces the `<img>` tag, commits + pushes. Includes the retroactive SVG conversion for `understand005_zh..009_zh.html` figures still referenced as PNG, plus understand012 figures 056-058.

### Already done in this session
- understand010 (Chapter 7): translated + all 4 figures (033-036) SVGified.
- understand011 (Chapter 8): translated + all 17 figures (037-053) SVGified.
- understand012 (Chapter 9): translated; figures 054-055 SVGified; 056-058 deferred to Phase 2.
- understand013 (Chapter 10) … understand020 (Appendix C): all translated (prose) in Phase 1.
- understand021 (Appendix D): SKELETON + D.1 + D.2.1.1 only. The rest is BLOCKED — see below.
- Navigation/cross-reference sweep: all translated chapters now link to `_zh.html` siblings (only legitimate placeholders remain, pointing to as-yet-untranslated appendices).
- Root `index.html` no longer auto-redirects; `translated_html/index.html` links all completed chapters and appendices.

### Status of remaining work (paused 2026-05-22)
- **Appendix D (`understand021_zh.html`)** — PARTIAL. Skeleton committed; D.1 + D.2.1.1 translated. D.2.1.2 through D.6 remain in raw English in the target file. Sub-agent translation of this 4317-line file has FAILED SILENTLY four times (claude-sonnet-4.6 ×3 and claude-opus-4.7 ×1) with `total_turns: 0` and no commit. The reliable strategy is to translate manually in the main session, one H4 function per commit, but the per-turn cost is high. Resume after the simpler appendices are done, OR consider a script that pre-extracts prose-only segments for translation.
- **Appendix E (`understand022.html`, 619 lines, 27 PRE)** — NOT STARTED. Sub-agent `xlate-022` was launched then stopped at user's request without producing any file. Should be tried again with a fresh session; this file is small enough that sub-agents usually complete it.
- **Appendices F, G, H, I, J, K (`understand023.html`–`understand028.html`)** — NOT STARTED. Sizes: F=653, G=427, H=1571, I=531, J=1217, K=1443 lines.
- **Phase 2 SVG conversion**: ALL of Phase 2 still pending — figures 056-058 in understand012_zh, retroactive PNG→SVG for understand005-009 (28 figures), and figures in understand013-020 + appendices once those are translated.

### Sub-agent silent-fail mitigation (lessons learned)
- Sub-agents (`general-purpose`, sonnet or opus) become unable to complete work on translated files once the target is large (>~3000 lines) or when the sub-agent's `edit` budget is exhausted by many small interleaved changes.
- For small source files (<1000 lines): single sub-agent works reliably (proven on understand010-020).
- For medium source files (1000-2000 lines): split by H3 subsection.
- For large source files (>3000 lines, like App D): hand-translate in main session, one H4 function per commit. Sub-agents are NOT reliable.

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

### Navigation / cross-reference links
- All `href` to sibling chapters MUST point to the `_zh.html` version when that chapter has been translated.
- Nav blocks (Prev / Next / Up / TOC) and inline cross-references (e.g. `<a href="understandNNN.html#anchor">第 X.Y 节</a>`) both follow this rule.
- If the target chapter is not yet translated, keep `../original_html/understandNNN.html#anchor` as a placeholder. When that chapter is later translated, sweep all files that reference it and rewrite to `understandNNN_zh.html#anchor`.
- After translating each new chapter, run a quick scan:
  `grep -rE 'href="(\.\./original_html/)?understand[0-9]+\.html' translated_html/`
  and rewrite any link whose target is now translated.

### Appendix translation: prose vs. code
The appendices (`understand019_zh.html` onward, App A–K) are mostly annotated kernel source listings: large `<pre class="verbatim">` blocks of C code interleaved with bulleted prose that explains each numbered region. Rules:

1. **Code blocks (`<PRE>` / `<pre class="verbatim">`) are NOT translated.** Keep them byte-for-byte identical to the source: identifiers, keywords, string literals, whitespace, and inline `/* … */` or `//` comments INSIDE the code stay in English.
2. **Only prose between code blocks is translated.** This includes paragraphs, section headings, function descriptions, and the bulleted line-by-line explanations that reference line numbers in the preceding `<PRE>` block.
3. **C identifiers in prose stay English.** Wrap them in `<tt>…</tt>` (or keep existing `<tt>`/`<code>` markup) — e.g. translate "The function `kmem_cache_create` allocates …" as "函数 `kmem_cache_create` 用于分配 …".
4. **Verification:** after translating an appendix, run `grep -c '<PRE' original_html/understandNNN.html` and the same on the `_zh` file — counts must match. Diff the `<PRE>` blocks if uncertain.
5. **Sub-agent prompts MUST repeat rules 1–4 verbatim** so each fresh context window applies them consistently.

### Appendix D retry strategy (xlate-021)
Previous two attempts on `understand021.html` (4317 lines) silently produced no output. For the retry:
- First sub-agent: only `cp original_html/understand021.html translated_html/understand021_zh.html`, swap the CSS link to `understand_zh.css`, translate the `<title>` and top-level headings, commit the skeleton.
- Then one sub-agent per H2 section (D.1 … D.6): translate prose only in that section's range, leave `<PRE>` blocks untouched, commit + push.
- This keeps each sub-agent's working set well below the context limit.
