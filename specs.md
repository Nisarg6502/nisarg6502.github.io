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
Case files are a horizontal carousel (`#caseViewport` > `#caseTrack` >
`.case-file-entry` slides, one visible at a time, switched via tabs in
`#caseTabs` + prev/next arrows + a "NN / NN" counter). To add one:
1. Duplicate a `.case-tab-btn` inside `#caseTabs`, bump `data-case` index
   and the letter/name shown.
2. Duplicate a `.case-file-entry` inside `#caseTrack`, bump `data-case` to
   match and the exhibit letter. Fill in `.case-name`, `.case-desc`
   paragraphs, and `.spec-sheet` rows. Omit `.demo-box` entirely if the
   project has no interactive demo — only the Llama/PII project has one.
3. Nothing else to touch — the carousel JS (search "case carousel" in the
   `<script>` block) reads entry/tab count from the DOM automatically.

External links (GitHub, HF, etc.) go in the `LINKS` object near the top of
the `<script>` block, then reference by element id like the existing
`ghLink`/`hfLink`/`cpGhLink` pattern — give new link spans unique ids since
ids must be unique per page.

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

## Open items
- Counterpoint (Exhibit B) has no live demo yet (not deployed). No "Live
  demo"/"Live app" spec row was added for it — add one + wire a URL into
  `LINKS` when it's deployed.
- Resume PDF (`Nisarg Kudgunti Resume.pdf`) is the source of truth for work
  experience wording — when it's updated again, diff it against the docket
  section bullets in `#experience`.

## Session log
- 2026-07-13: Removed the floating background token-particle canvas
  animation (hero). Reordered sections (Record before Case Files).
  Restructured Case Files section to support multiple project entries
  (`.case-file-entry` + `.exhibit-tag`, currently only Exhibit A / Llama
  PII redactor filled in). Replaced meaningless "PERSONNEL FILE NO. 0150M"
  hero tag with "FILE STATUS: ACTIVE — OPEN TO OFFERS". Added themed screen
  loader + staggered hero entrance animation. Removed dead `.redact-hint`
  CSS (was unused in markup). Added `.gitignore` and this file.
- 2026-07-13 (later): Replaced resume PDF with updated version; synced
  `#experience` docket bullets to match it (AI Intern bullets rewritten,
  Associate AI Engineer's last two bullets reworded/merged). Added Exhibit B
  case file: Counterpoint (corporate contradiction detector over SEC
  filings — LangGraph agentic hybrid RAG, Neo4j, Qdrant). Added Langfuse tag
  to AI systems and Qdrant/Neo4j tags to Data, matching resume + new
  project's stack.
- 2026-07-13 (again): Wired Counterpoint's GitHub URL into `LINKS`. Reworked
  Case Files from a vertical stack (was making the page too long) into a
  horizontal carousel — one exhibit visible at a time, tabs + prev/next
  arrows + "NN / NN" counter, viewport height animates to match the active
  slide. Filled the previously-empty hero right column with a static
  `.hero-visual` dossier-stamp card (redacted-line mockup + exhibit/role
  stat counts) — no motion beyond a one-time fade-in, per "no floating
  stuff" from earlier. Redesigned `.index-card` (skills section): ghost
  watermark number, two-letter category code badge, hover lift + shadow —
  was flagged as "too bland."
- 2026-07-13 (yet again): `.hero-visual` gray skeleton-style bars read as an
  unfinished loading state, not a redacted document — replaced with real
  "case log" text using the site's `.redact`/`.rtext`/`.rbar` hover-reveal
  component (previously defined in CSS but unused anywhere in markup).
  Reworded the About section's closing line ("that project sits at the
  center of this page") since it was written when there was only one case
  file — now references both Llama-PII and Counterpoint.
- 2026-07-13 (PR review pass): Opened branch `feat/counterpoint-case-carousel-ux`,
  had a subagent independently review the diff. Fixed 3 real bugs it found
  in the case carousel JS (search "case carousel" in `<script>`): (1)
  inactive slides were still keyboard/screen-reader reachable despite being
  visually clipped — now toggle `aria-hidden`/`inert` on non-active
  `.case-file-entry` elements; (2) `.is-active` on `.case-file-entry` was
  dead (no CSS targeted it) — now used as the hook for the aria-hidden/inert
  toggle above, and also to force-mark that slide's `.reveal` children
  `is-visible` immediately on switch (previously they could pop in a beat
  late since `IntersectionObserver` wasn't triggering on the horizontally
  offset, `overflow:hidden`-clipped inactive slide); (3) `setHeight()` never
  recomputed after web fonts swapped in — added a `document.fonts.ready`
  listener alongside the existing resize listener.
- 2026-07-22: Added Exhibit C: PayWise (LangGraph credit card optimizer,
  deployed on Cloud Run). Has a live app + API docs, so its `.demo-box` is a
  link pair (`pwAppLink`, `pwDocsLink`) instead of an interactive demo. Source
  wired as `pwGhLink`. All three URLs added to `LINKS`
  (`paywiseApp`/`paywiseDocs`/`paywiseRepo`). Added `Next.js` tag to
  Frameworks and `PostgreSQL` tag to Data in the skills index; LangGraph,
  FastAPI, Qdrant, GCP were already covered by existing tags.
