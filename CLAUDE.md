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
   (React, Vue, Angular, jQuery, Tailwind, etc.) and no build step (no
   bundlers, transpilers, package.json, npm install, etc.). External library
   `<script>`/`<link>` tags are fine per constraint 1, but the project's own
   code must not depend on a framework or build tooling.

When implementing any feature, keep all markup, styles, and scripts inside
`index.html`.

## Working conventions
- Before implementing any non-trivial feature, ask clarifying questions about
  scope, edge cases, and constraints first — don't propose a plan until
  you've asked.

## Feature Plan

Status legend: `[ ]` not started, `[x]` done. Mark phases done or prune
finished detail as work progresses — keep this section skimmable, not a
full spec.

### Data model (shared across all phases)
- `TOOL_REGISTRY` entry: `{ id, title, description, templateId, init }`
- `FLUID_PRESETS`: `{ water: 1000, air: 1.225, oil: 900 }` (kg/m³) + custom
  density option
- Physics module: `areaFromDiameter(d)`, `continuityVelocity(A1,V1,A2)`,
  `solveContinuity({A1,V1,A2,V2})`, `bernoulliTerms(rho,V,h,g)`

### Key flows (shared across all phases)
- **Navigation**: hashchange → look up registry entry by id → destroy
  currently mounted tool → clone its `<template>` into `#tool-mount` → call
  `init(root)`
- **Mount/unmount**: `init(root)` wires its own controls + rAF loop,
  returns `{ destroy() }`; router always destroys before remounting
- **Input → viz**: a tool's slider/dropdown changes update closures read by
  `createPipeFlowViz`'s `getProfile()`/`getInletVelocity()`, so the shared
  animation reflects current state with no extra glue
- **Theme**: toggle → `data-theme` attr + `localStorage['fdl-theme']` → all
  CSS via custom properties + canvas colors re-read at draw time

### Visual design — Direction A: "Technical Lab" (chosen)
- **Fonts**: `IBM Plex Sans` (UI/headings), `IBM Plex Mono` (data readouts,
  values, labels) — via Google Fonts `<link>` in `<head>`.
- **Palette (light)**: bg `#fafbfc`, fg `#1a2027`, muted `#5b6670`, card bg
  `#ffffff`, border `#d7dee5`, accent primary (teal) `#3b9aa8`, accent
  secondary (amber) `#b5791f`.
- **Palette (dark)**: bg `#14181c`, fg `#e7ecf0`, muted `#8a97a1`, card bg
  `#1b2126`, border `#2b333a`, same accent hex values (kept unchanged across
  themes — sufficiently legible on both).
- **Look**: rectangular cards, minimal border-radius (~4–5px), 1px borders,
  thin-line SVG icons, faint dot/grid "blueprint" texture behind content,
  monospace used specifically for numeric readouts/values to separate data
  from UI chrome.
- Reference mockup (for tone/layout, not exact markup):
  https://claude.ai/code/artifact/97ef6c25-36ae-41d1-a290-0468d580b4ee
  (artboard "A — Technical Lab")

### Phases
- `[x]` **Phase 1 — Shell & routing**: header (title, Home button, theme
  toggle), home/tool view containers, empty `<template>` stubs, tool
  registry + hash router + grid renderer, theme toggle via CSS vars +
  localStorage. Proven with 2 placeholder templates.
- `[x]` **Phase 2 — Shared physics + pipe-flow viz**: physics module (pure
  functions, no DOM); `createPipeFlowViz(canvas, options)` factory
  (smoothstep-tapered `radiusFn`, particles advancing by
  `velocity/area(x)`, speed-colored draw loop); sanity-checked against a
  throwaway profile — not yet wired into either tool.
- `[ ]` **Phase 3 — Venturi Tube tool**: template (mode toggle
  convergent/divergent, throat-diameter + inlet-velocity sliders, fluid
  dropdown + custom density, canvas, live velocity/pressure readouts,
  theory panel with continuity + Bernoulli equations, h1=h2=0 assumption
  stated). `initVenturiTool` wires physics + shared viz + inputs.
- `[ ]` **Phase 4 — Bernoulli/Continuity tool**: template (reuses shared
  viz with a simple two-point taper, elevation h1/h2 numeric inputs shown
  as flat-pipe side labels, live Bernoulli term readout table, continuity
  solve-for-unknowns form with exactly-one-blank validation, own theory
  panel). `initBernoulliTool` wires physics + shared viz + solver.
- `[ ]` **Phase 5 — Polish**: correct teardown on rapid tool switching (no
  leaked rAF loops), responsive layout at narrow widths, dark-mode
  canvas/particle contrast check, initial-hash-on-load deep linking.
