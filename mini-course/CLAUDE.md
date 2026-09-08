# CLAUDE.md — Mini-Course Site

Guidance for Claude Code (and any other contributor) working in this
subfolder. This file is scoped to `mini-course/` only — it does not modify
or replace the repo-root `CLAUDE.md`, which documents a separate, unrelated
project.

> **Status: draft, pending slides.** The user has not yet shared the
> source slide images. Everything below is a placeholder skeleton based on
> the general brief ("turn 3-5 slides into a mini-course teaching site,
> lesson-style, with navigation between sections, one-time build — not a
> visitor-upload feature"). Do not start implementing from this file alone;
> revisit both sections once slides are shared and the remaining interview
> questions (topic, audience level, tone, whether quizzes/exercises are
> wanted, navigation style — sidebar vs. prev/next) are answered.

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
- `LESSONS`: ordered array of `{ id, navLabel, title, bodyHtml }`, one
  entry per slide — populated once slides are shared and their content is
  written up as lesson prose.
- No other persisted state expected (no backend, no localStorage need
  identified yet — add if a "resume where you left off" requirement
  emerges from the interview).

### Key flows
- **Navigation**: nav control (sidebar list or prev/next buttons — TBD)
  → show the matching `LESSONS` entry, hide the rest → update active-nav
  highlighting.
- **Content authoring**: each slide image is read once, its content
  rewritten into a `bodyHtml` lesson section (not a screenshot, not a
  bullet dump) and added to `LESSONS` in slide order.

### Phases
- `[ ]` **Phase 0 — Intake**: receive the slide images, finish the
  requirements interview (topic, audience, tone, nav style, whether
  exercises/quizzes are wanted, how many lessons exactly). Fill in the
  real data model and phase list below based on the answers — this
  skeleton gets replaced, not just appended to.
- `[ ]` **Phase 1 — Shell & navigation**: page scaffold, `LESSONS`
  registry (stub content), nav UI, section switching wired up and proven
  with placeholder text before real content goes in.
- `[ ]` **Phase 2 — Lesson content**: write up each slide's content as a
  lesson section, in order.
- `[ ]` **Phase 3 — Polish**: responsive check, visual pass, verify
  navigation and deep-linking (if used) all work end to end.
