# CLAUDE.md

Guidance for Claude Code (and any other contributor) working in this repository.

## Stack & Conventions

**Hard constraints — do not violate these:**

1. **Single-file project.** The entire project must live in one `index.html`
   file. All CSS and JavaScript must be inlined in that file using `<style>`
   and `<script>` tags — never in separate `.css` or `.js` files, and never
   split across additional `.html` pages. Linking to external images, CSS
   libraries, and JavaScript libraries (e.g. via CDN `<link>`/`<script src>`
   tags) is allowed. This constraint exists so the finished project can be
   copy-pasted as a single file for sharing in class and on single-file code
   platforms (e.g. CodePen, JSFiddle).
2. **Vanilla only.** Use plain HTML, CSS, and JavaScript only. No frameworks
   (React, Vue, Angular, jQuery, etc.) and no build step (no
   bundlers, transpilers, package.json, npm install, etc.). External library
   `<script>`/`<link>` tags are fine per constraint 1, but the project's own
   code must not depend on a framework or build tooling.

When implementing any feature, keep all markup, styles, and scripts inside
`index.html`.

## Tech stack (hard constraints — do not deviate)
- Vanilla HTML, CSS, and JavaScript only. No React, Vue, or any JS framework.
- Tailwind CSS for all styling (via CDN only).
- No backend, no database. Fully static site.
- A toggle for light and dark theme, with the choice remembered
  across visits.

## Working conventions
- Before implementing any non-trivial feature, ask clarifying questions about
  scope, edge cases, and constraints first — don't propose a plan until
  you've asked.

## Feature Plan

Status legend: `[ ]` not started, `[x]` done. Mark phases done or prune
finished detail as work progresses — keep this section skimmable, not a
full spec.

**Product**: a phone-first portal of small tools and lessons for students
at the P.A. Hilton S220 Rankine Cycle Steam Turbine rig (source: the S220
manual, S220/E/5/043). The collection grows over time; phase one ships
exactly two items: one lesson and one tool. (This restarts the project —
the earlier Venturi/Bernoulli portal was discarded; it lives in git
history.)

### Data model
- `CATALOG` entry: `{ id, type: 'tool'|'lesson', title, summary, topics[],
  keywords[], templateId, init }`. Adding an item = one entry + one
  `<template>`.
- `S220` constants: brake effective radius 0.023 m, rotor I = 49.7e-6
  kg·m², heat loss Qr = 3.2·(t5 − ta) W, cooling water Cp 4.18 kJ/kg·K,
  overspeed 40 000 rpm.
- `MANUAL_DATA`: the manual's sample tests (A–D, F–K, …) with errata
  corrected; each correction carries a short note shown in the UI where
  it differs from the printed manual.
- `Perf`: pure functions (abs pressure, torque, power, steam flow, SSC,
  cooling-water heat, heat loss) + `computeReading(row)` → all derived
  values, `null` where an input is missing.
- `Steam`: wrapper over `iapws-if97@2.2.0` (MIT, zero deps, pure ESM),
  dynamically imported from pinned jsDelivr files `…/dist/index.js`.
  Units: bar abs, °C, kJ/kg, kJ/(kg·K). `pt/ph/ps/px/tx/sat`;
  `Steam.status` loading|ready|failed + `steamstatus` event. Vetted
  against IAPWS verification tables and the manual's worked examples.
  Calls throw `SteamUnavailableError` when not loaded; catalogue entries
  with `needs: ['steam']` get a loading/failed banner — never wrong
  numbers.
- `Store` (namespaced `s220:` localStorage, failure-tolerant), `Csv`
  (`build`, `download` with BOM + formula-injection guard).

### Key flows
- **Navigation**: hash routes `#/tool/<id>`, `#/lesson/<id>`; home keeps
  search + filter state in the hash (`#/?q=…&type=…&topic=…`). Router
  calls the mounted item's `destroy()` before cloning the next template
  and calling `init(root)`; unknown hash → home.
- **Home**: card grid + text search (title/summary/keywords) + filter
  chips (type, topic).
- **Theme**: `.dark` class on `<html>`, set before first paint; choice in
  `localStorage`, defaulting to the OS preference.
- **Persistence**: calculator edits auto-save to `localStorage`, with
  "Reset to manual data"; CSV export of inputs + results.

### Visual design — Direction B: "Engine Room" (chosen)
- Tailwind v4 in-browser build, pinned (`@tailwindcss/browser@4.1.18` on
  jsDelivr); config is CSS-first in `<style type="text/tailwindcss">`
  (`@theme inline` maps tokens to CSS vars, `@custom-variant dark` on
  `.dark`). v4 renames `rounded` — use explicit `rounded-[2px]`. (v3's
  cdn.tailwindcss.com is unreachable from the dev sandbox, so it can't be
  tested.) Fonts (Google Fonts):
  `Barlow Condensed` (headings, uppercase, tracked), `Barlow` (body),
  `JetBrains Mono` (readouts, inputs, codes).
- Dark: bg `#16181a`, surface `#202326`, border `#34383c`, fg `#eceff1`,
  muted `#a3aab0`, accent safety-orange `#f08a24` (text `#f5a04f`),
  secondary steel-blue `#7fb2e5`. Light theme: derived equivalents with
  the same accents, contrast-checked.
- Look: near-square corners (2px), 1px borders, orange header rule,
  large 48px inputs and ≥44px touch targets, monospace numeric readouts
  in ruled result rows.
- Mockups: https://claude.ai/artifact/AvrEqTitS76Jmbw9GJD8bR (artboard
  "B — Engine Room").

### Phases
- `[x]` **Phase 1 — Shell**: Tailwind config, theme toggle, catalogue,
  router, home grid with search + filters; proven with 2 placeholder
  entries (each runs a timer to prove `destroy()` on navigation).
- `[x]` **Phase 2 — Shared modules**: vet + wrap steam library, S220
  constants, `MANUAL_DATA` with errata notes, storage + CSV helpers.
- `[x]` **Phase 3 — Turbine Performance Calculator** (`initTurbineTool`):
  single-reading mode (defaults to manual B.4) with results, sanity
  warnings and worked steps; test-table mode (manual tests A–K + "My
  readings") with responsive reading rows (results under inputs on
  phones), add/remove rows, errata box, Chart.js 4.5.1 (jsDelivr, loaded
  on demand via `Libs.load`) torque/power/SSC-vs-speed charts, one chart
  per measure (no dual axes). Series colour `--series` validated per
  theme; charts restyle on `themechange`. Auto-save under
  `perf:single|mode|test|tests`; "Reset to manual data"; CSV export.
- `[x]` **Phase 4 — Rankine Cycle Explainer** (`initRankineLesson`):
  Explore / Walkthrough / Quiz tabs (shared `wireTabs`). SVG schematic
  (`rankineSchematic`) with 7 tappable components (keyboard too) and a
  pausable flow animation (paused under reduced motion). 5-step
  walkthrough highlights schematic parts + T–s segments; T–s diagram
  (`tsDiagram`, zoomed 40–180 °C, dome clipped) and state list from
  `rankineCycle(5.6, 0.5, η=0.134)` — note the actual turbine exit is
  slightly superheated. 7-question quiz with explanations and score.
  T–s parts show loading/failed messages; the rest works without steam.
- `[x]` **Phase 5 — Polish**: audited every view at 320/375/768/1280px in
  both themes with axe-core (WCAG 2.2 AA) — clean; no horizontal scroll;
  light `--accent-ink` darkened to `#a85500` for 4.5:1 text contrast;
  inputs get a 2px `outline-solid` focus ring (v4 `outline-none` kills
  outline style); back link and browser Back keep home filters; no leaked
  charts/listeners after rapid switching; works with all CDNs blocked
  (SVG presentation-attribute fallbacks) and with storage blocked.

**Phase one complete.** `index.html` ships the shell, the Turbine
Performance Calculator and the Rankine Cycle Explainer. (Verification
used Playwright + axe-core run outside the repo — no tooling is committed,
per the single-file constraint.)

**Later phases (not in phase one)**: isentropic/thermal efficiency,
retardation (friction) test, steam quality by throttling, heat/power
ratio.
