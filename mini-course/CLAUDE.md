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

6 sections total: 4 lessons + 1 interactive simulator + 1 final quiz,
navigated the same way (sidebar + Prev/Next), all via `LESSONS`.

### Data model
- `COMPLEXITY_CLASSES`: `{ key, label, color, fn(n) }` × 6 (O(1) through
  O(2ⁿ)) — shared by Lesson 2's text-only ranking and the simulator's
  live bars, so the growth functions live in one place.
- `LESSONS`: ordered array of sections, one of three `kind`s:
  - `"lesson"`: `{ id, navLabel, title, kicker, body: [{heading?, paragraphs?, code?, list?}], checkIn }`
  - `"simulator"`: same shape as `"lesson"` plus rendered via
    `renderSimulator()` (interactive controls, not static body markup).
  - `"final"`: `{ id, navLabel, title, kicker, intro, questions: [{question, options, correctIndex, explanation}] }`
    (3 questions).
  - `checkIn` (one per lesson/simulator section — not an array):
    `{ type: "mc", question, options, correctIndex, explanation }` or
    `{ type: "short", question, acceptableContains: [...], explanation }`.
- Runtime state (in-memory JS, not persisted): `currentIndex` into
  `LESSONS`; `sectionState[id]` holds each section's check-in answer
  (`{selected, submitted}` for mc, `{text, submitted}` for short-answer)
  or, for the final section, an array of 3 such states; `simState`
  holds the simulator's current `n` and which complexity classes are
  toggled on. Nothing persisted across a reload — not needed for a
  one-time course site.

### Key flows
- **Navigation**: sidebar list (numbered step + label) and Prev/Next
  buttons both call `goToLesson(index)` → re-renders the whole section
  pane, sidebar active/done states, and the top progress-dot strip.
  Continuing past a section's check-in is always allowed regardless of
  correctness (Prev/Next are never gated on quiz state).
- **Check-in (single question per lesson/simulator section)**: answering
  → "Check answer" reveals correct/incorrect styling (mc) or a
  substring-match grade against `acceptableContains` (short-answer),
  plus a one-line explanation either way. Shared `buildQuestionBlock()`
  renders both the per-section check-in and the final quiz's 3
  questions so the interaction is identical everywhere.
- **Simulator**: a range slider (`n` from 1–30) and one checkbox per
  complexity class drive `updateSimulatorOutput()`, which patches just
  the bar widths/values and the note text in place — no full
  `renderLesson()` call on every slider tick, so the interaction stays
  smooth. Bar width is `log10(value+1)` scaled against the largest
  currently-visible value, since O(2ⁿ) dwarfs the others on a linear
  scale.
- **Final quiz**: 3 independently-checkable questions (same
  check/feedback pattern as above); once all 3 are submitted, a
  "You got X out of 3 correct" summary appears beneath them.

### Phases
- `[x]` **Phase 0 — Intake**: no slides available; topic, audience, and
  scope decided per the user's answers above.
- `[x]` **Phase 1 — Shell & navigation**: `index.html` built in the
  chosen Direction B look, `LESSONS` registry with all 4 lessons' real
  content, sidebar + progress dots + Prev/Next wired to `goToLesson`.
- `[x]` **Phase 2 — Check-ins**: per-section single check-in question
  (mc or short-answer) replacing the earlier multi-question per-lesson
  quiz, inline in the same view, non-blocking for navigation.
- `[x]` **Phase 3 — Simulator section**: interactive growth-rate
  comparison tool (slider + toggles + live bar chart) as its own
  section between the lessons and the final quiz.
- `[x]` **Phase 4 — Final quiz**: 3-question comprehensive quiz as the
  last section, reusing the check-in question renderer, with a
  completion summary.
- `[x]` **Phase 5 — Polish**: verified with Playwright — all 6 sections
  reachable with no page reload, short-answer and mc check-ins grade
  correctly, simulator slider/toggles update live without losing quiz
  state elsewhere on the page, final quiz per-question feedback +
  summary, no horizontal overflow at 375px or 1280px. Google Fonts
  failed to load in the sandboxed test environment (network policy);
  confirmed the system-ui/sans-serif/monospace fallback stacks render
  cleanly either way — not expected in a normal browser.
