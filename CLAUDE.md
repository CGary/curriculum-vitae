# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

Single-page CV rendered with **Preact + HTM loaded from CDN** — no bundler, no `package.json`, no build step. The entire app lives in `index.html`:

- Preact UMD + HTM UMD are pulled from `unpkg.com` via `<script>` tags. The `hooks` and `compat` CDN scripts are intentionally commented out (commit `5ca15f0`); only `preact.umd.js` and `htm.umd.js` are active. Don't re-enable them unless a component actually needs hooks/compat.
- The `<script type="module">` block at the bottom of `index.html` defines `cvData` (the source of truth for rendered content) and a small set of functional components: `Header`, `SobreMi`, `Experiencia`, `Educacion`, `Habilidades`, `Intereses`, composed by `CV`.
- HTM is bound via `window.htm.bind(h)` — components use tagged-template syntax (`html\`...\``), not JSX. There is no transpilation step that could turn JSX into `h()` calls.

### Running it

Open `index.html` directly in a browser, or serve the directory statically (e.g. `python -m http.server`, `bunx serve`). There is no dev server, no test runner, no lint config — if you add tooling, confirm with the user first; the explicit choice here is "zero build".

## The `data/` directory is the source of truth

`data/` is gitignored (see `.gitignore` — though note it's currently untracked rather than ignored by pattern; `git status` shows `?? data/`). It contains the raw inputs that the CV in `index.html` should reflect:

- `cv-ia.md` — narrative profile content
- `Empleo Liderazgo Técnico 2026.md` — strategic ATS-optimization guidance the CV should satisfy
- `pag1.txt` … `pag6.txt` — paginated source material

`prompt.md` documents the standing instruction: refactor `index.html` content (the `cvData` object and components) to reflect what's in `data/`, prioritize quantifiable achievements and architectural decisions, and ensure the result passes modern ATS filters while staying ≤2 pages when printed to PDF.

When the user asks to "update the CV": read `data/` first, then edit the `cvData` literal and/or components in `index.html`. Do not invent content that isn't in `data/`.

## Print/PDF is a first-class target

The CV must work in two modes:

- **Web**: modern, professional styling (current inline `<style>` block in `index.html`).
- **Print/PDF**: optimized via `@media print` — strip interactive affordances, fit within 2 pages, maximize legibility for recruiters.

If you touch styling, verify both modes. A `@media print` block is currently absent — adding/maintaining one is part of the project's intent per `prompt.md`.

## Conventions

- **Language**: All user-visible CV content is in Spanish. Component code, identifiers, and keys in `cvData` are also in Spanish (`habilidades`, `experiencia`, `logros`, `puesto`, etc.) — match this when extending.
- **Commits**: Conventional commits (`feat:`, `fix:`, `chore:`, `style:`). Recent history confirms the style. Per global rules: no AI attribution / `Co-Authored-By` lines.
- **No frameworks beyond Preact+HTM**: don't introduce React, a bundler, TypeScript, or a CSS framework without explicit user approval. The minimal-dependency stance is deliberate.
