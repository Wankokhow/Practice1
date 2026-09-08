# CLAUDE.md — Mini-Course Site

Guidance for Claude Code (and any other contributor) working in this
subfolder. This file is scoped to `mini-course/` only — it does not modify
or replace the repo-root `CLAUDE.md`, which documents a separate, unrelated
project.

> **Status: building.** No slides were available; the user asked instead
> for a mini-course authored from publicly available knowledge on a topic
> Claude picks. Topic: **Big-O Notation & Algorithm Complexity** — a
> well-documented CS fundamentals topic, good fit for the "some prior
> background" audience level the user specified. 4 lessons, each ending
> in a short self-check quiz, sections switched via JavaScript (no
> separate pages), in the "Modern Minimal" visual direction already
> chosen below.

## Stack & Conventions

**Hard constraints — do not violate these:**

1. **Single-file project.** The entire project — every lesson section and
   the navigation between them — must live in one `index.html` file inside
   `mini-course/`. All CSS and JavaScript must be inlined in that file
   using `<style>` and `<script>` tags — never in separate `.css` or `.js`
   files, and never split across additional `.html` pages. Linking to
   external images, CSS libraries, and JavaScript libraries (e.g. via CDN
   `<link>`/`<script src>` tags) is allowed. This constraint exists so the
   finished project can be copy-pasted as a single file for sharing in
   class and on single-file code platforms (e.g. CodePen, JSFiddle).
2. **Vanilla only.** Plain HTML/CSS/JS, no frameworks, no build step.

**Visual design — Direction B: "Modern Minimal" (chosen)**
- **Fonts**: `Space Grotesk` (headings), `Work Sans` (body/UI), `Space
  Mono` (numeric/code readouts) — via Google Fonts `<link>` in `<head>`.
- **Palette**: bg `#fbfbfc`, fg `#1c1f26`, muted `#5b6070` / `#8b909c`,
  border `#e5e7ec`, accent (indigo) `#3454d1`, accent hover `#26399c`,
  active-nav-item tint `#eef0fb`.
- **Look**: cool neutral background, generous whitespace, ~8-10px
  border-radius on chips/buttons/cards, thin 1px borders, no heavy
  shadows. Sidebar nav uses numbered circular step markers (outlined =
  upcoming, filled indigo = active) plus a top progress-dot strip.
  Primary nav action ("Next lesson") is a filled indigo pill button;
  "Previous" is plain muted text.
- Chosen from a 3-direction proposal (Academic Serif / Modern Minimal /
  Friendly Bold) — reference mockup (for tone/layout, not exact markup):
  https://claude.ai/code/artifact/fcf3c4eb-504a-4064-9d4a-c3ed09971d18
  (page "Design", artboard "Mini-Course Site — Modern Minimal"; the two
  unchosen directions are kept on that canvas's second page for
  reference).

**Folder structure**
- `mini-course/index.html` — the whole site.
- `mini-course/CLAUDE.md` — this file.
- Slide-derived images/diagrams (if any are reused as-is rather than
  redrawn), if needed: `mini-course/assets/`.

**Naming conventions**
- Section ids: `lesson-1`, `lesson-2`, … (kebab-case, ordinal).
- JS: camelCase for functions/vars; a single `LESSONS` registry array as
  the source of truth for section content + nav labels.

**Standing rules**
- This is a one-time build from specific, user-provided slides — no
  upload UI, no generic "paste your slides" feature.
- Content is written as prose lesson sections (intro, explanation,
  examples), not a slide-by-slide restatement — synthesize each slide's
  point into teaching text.
- Navigation between sections happens without a page reload (single page,
  JS-driven show/hide or hash routing).
- Before implementing any non-trivial feature or ambiguous slide
  interpretation, ask clarifying questions first rather than guessing.
- Check mobile/narrow-width layout before considering any phase done.

## Feature Plan

Status legend: `[ ]` not started, `[x]` done.

### Data model
- `LESSONS`: ordered array of
  `{ id, navLabel, title, kicker, body: [{heading?, paragraphs?, code?, list?}], quiz: [{question, options, correctIndex, explanation}] }`.
  4 entries, one per lesson (see Phases). `body` is a small block list
  (not a raw HTML blob) so paragraphs/code/lists render with consistent
  markup.
- Runtime state (in-memory JS, not persisted): `currentIndex` into
  `LESSONS`; per-lesson quiz state (`selectedOptionIndex`,
  `submitted`) held in a plain object keyed by lesson id, reset only on
  page reload — no localStorage (nothing here needs to survive a
  refresh for a one-time course site).

### Key flows
- **Navigation**: sidebar list (numbered step + label) and
  Prev/Next buttons at the bottom of the lesson pane both call one
  `goToLesson(index)` → updates `currentIndex`, re-renders the lesson
  pane and the sidebar's active/upcoming states, updates the top
  progress-dot strip. All via JS show/hide — no separate pages, no
  hash routing needed for a fixed 4-lesson course.
- **Quiz interaction**: selecting an option stores it in that lesson's
  quiz state; "Check answer" reveals correct/incorrect styling per
  option plus a one-line explanation; re-selecting before checking is
  allowed, changing the answer after checking resets to unchecked.

### Phases
- `[x]` **Phase 0 — Intake**: no slides available; topic, audience, and
  scope decided per the user's answers above.
- `[x]` **Phase 1 — Shell & navigation**: `index.html` built in the
  chosen Direction B look, `LESSONS` registry with all 4 lessons' real
  content, sidebar + progress dots + Prev/Next wired to `goToLesson`.
  Verified with Playwright: click-through of all 4 lessons, no page
  reload, Prev/Next disable correctly at the first/last lesson.
- `[x]` **Phase 2 — Quizzes**: quiz rendering + check/feedback logic
  for all 4 lessons' question sets, verified interactively (select →
  check → correct/incorrect styling + explanation).
- `[x]` **Phase 3 — Polish**: verified at 375px width (sidebar
  collapses to a horizontal scrollable step strip, no horizontal
  overflow) and at 1280px desktop. Google Fonts failed to load in the
  sandboxed test environment (network policy), confirmed the
  system-ui/sans-serif/monospace fallback stacks render cleanly either
  way — not expected to recur in a normal browser.
