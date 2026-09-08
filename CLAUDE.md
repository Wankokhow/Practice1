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
