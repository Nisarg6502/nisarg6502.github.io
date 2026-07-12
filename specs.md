# specs.md — working notes for future sessions

Read this first instead of the whole codebase. Update it whenever you make a
structural decision worth remembering. Keep it short.

## What this is
Single-file static portfolio site (`index.html` — all CSS/JS inline, no build
step, deployed via GitHub Pages from `main`). Repo:
github.com/Nisarg6502/nisarg6502.github.io

Theme: redacted case-file / dossier aesthetic (paper background, mono/serif
type, redaction bar wipe effects, "exhibit"/"docket"/"file" language).

## Section order (top to bottom, matches nav)
1. Hero (`#hero`)
2. About (`#about`) — eyebrow "01 / SUBJECT"
3. Experience (`#experience`) — eyebrow "02 / RECORD" — MUST stay before Case Files
4. Case Files (`#cases`) — eyebrow "03 / EXHIBITS" — projects, see below
5. Skills (`#skills`) — eyebrow "04 / INDEX"
6. Achievements (`#records`) — eyebrow "05 / ADDITIONAL RECORDS"
7. Contact (`#contact`) — eyebrow "06 / REQUEST ACCESS"

User explicitly wants Record (experience) before Case Files. Don't reorder
without asking.

## Adding a new project (case file)
Inside `<section id="cases">`, duplicate one `.case-file-entry` block.
Bump the exhibit letter (`EXHIBIT B`, `EXHIBIT C`, ...). Fill in
`.case-name`, `.case-desc` paragraphs, and `.spec-sheet` rows. Omit
`.demo-box` entirely if the project has no interactive demo — it's optional,
only the Llama/PII project has one. External links (GitHub, HF, etc.) go in
the `LINKS` object near the top of the `<script>` block at the bottom, then
reference by element id like the existing `ghLink`/`hfLink` pattern — give
new link spans unique ids (`ghLink3`, etc.) since ids must be unique per page.

## Design tokens
All in `:root` at top of `<style>`: `--paper`, `--ink`, `--accent` (rust red),
spacing scale `--s1`..`--s7`, fonts `--serif`/`--sans`/`--mono`. Reuse these,
don't hardcode new colors/spacing.

## Conventions
- No build tooling. Everything inline in `index.html`. Keep it that way
  unless the file becomes genuinely unwieldy (it's ~1300 lines now, fine).
- `.reveal` class + IntersectionObserver = scroll-in fade/slide, used on
  most section content blocks.
- `.spotlight` class = radial cursor-follow hover glow, used on cards.
- Respect `prefers-reduced-motion` — every animation added must have a
  reduced-motion fallback in the media query near the end of `<style>`.
- Screen loader (`#loader`) shows on every load, dossier-themed ("Compiling
  dossier…"), min ~650ms display, hides on `window.load` + `document.fonts.ready`,
  4s hard fallback so it can never get stuck. Hero content (`.hero-inner > *`)
  fades in staggered right after loader closes, via `body.loaded` class.

## Known environment quirk (not a code bug)
The in-session browser preview tool's screenshot/zoom sometimes hangs and
IntersectionObserver doesn't fire on programmatic (non-real) scrolls in that
tool. Verified via `get_page_text`/`read_page`/manual `dispatchEvent('scroll')`
that the actual reveal/nav-sync logic works correctly — it's a tooling
limitation, not a site bug. Don't waste time chasing it again; just verify
via DOM queries instead of screenshots if it recurs.

## Session log
- 2026-07-13: Removed the floating background token-particle canvas
  animation (hero). Reordered sections (Record before Case Files).
  Restructured Case Files section to support multiple project entries
  (`.case-file-entry` + `.exhibit-tag`, currently only Exhibit A / Llama
  PII redactor filled in). Replaced meaningless "PERSONNEL FILE NO. 0150M"
  hero tag with "FILE STATUS: ACTIVE — OPEN TO OFFERS". Added themed screen
  loader + staggered hero entrance animation. Removed dead `.redact-hint`
  CSS (was unused in markup). Added `.gitignore` and this file.
