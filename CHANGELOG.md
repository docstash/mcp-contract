# Changelog

Changes to the DocStash MCP server's agent-facing contract — the tools (their
descriptions and input schemas) and the always-on server `instructions` — recorded
alongside the contract they describe. The current version is reported as
`serverInfo.version` in the MCP `initialize` handshake.

The format follows [Keep a Changelog](https://keepachangelog.com/), and versioning is
[semantic](https://semver.org/):

- **MAJOR** — a breaking change to a tool's name, its arguments, or a required input.
- **MINOR** — a new tool, or a new capability / expanded contract on an existing one.
- **PATCH** — wording, clarity, or de-duplication that does not change behavior.

## [4.2.4] — 2026-10-28

### Changed

- **`set_access` is marked `openWorldHint: true`.** Granting access emails the
  person a link, which reaches someone outside DocStash. Annotation only; the
  tool behaves the same.

## [4.2.3] — 2026-10-27

### Removed

- **Retired tool names `create_docx`, `show_docx` and `show_sheet` are no longer
  accepted.** None is advertised, and none has been called since October 19. A
  call to one is now an unknown tool, like every other retired name.

## [4.2.2] — 2026-10-25

### Changed

- **`set_access` shares with anyone by email, account or not.** DocStash emails the
  person a link; without an account they view the doc in their browser from it and
  sign up to edit or download. The result's `say` reports whether the email went
  out.

## [4.2.1] — 2026-10-23

### Removed

- **Retired tool names `manage_document`, `manage_sharing`, `create_app`,
  `edit_app`, `create_sheet`, `edit_sheet` and `edit_docx` are no longer
  accepted.** None had been advertised since they were retired, and none has been
  called since October 14. A call to one is now an unknown tool. `create_docx`
  stays callable and still answers with the tools to use instead.
- **`show_docx` and `show_sheet` are retired.** DocStash no longer opens Word or
  spreadsheet documents; existing ones are download-only from the app. Both are
  no longer advertised; a call to either answers with that, like `create_docx`.
  The inline preview's spreadsheet bundle is gone: a `ui://docstash/sheet` widget
  resolves to the document bundle, which says the doc can only be downloaded.

## [4.2.0] — 2026-10-21

### Added

- **`get_file` takes `link: true` and can answer with `fileUrl`.** The inline
  preview asks for a short-lived signed storage link and fetches the document's
  source itself, so no document bytes ride the tool result. The server only
  returns the link when the widget's CSP reaches that storage host; otherwise,
  and whenever `link` is absent, it inlines `dataUrl` as before. A widget whose
  direct fetch fails asks again without `link`. PDFs still answer with `pdfUrl`.

### Changed

- **`create_text` takes the file's `extension` in place of `type`, `language` and
  `filename`.** Its input is `{ name?, slug?, extension?, content, description? }`.
  `extension: 'md'` (or `'markdown'`) makes a Markdown doc; any other extension makes
  a code or data file with that extension (`'py'`, `'csv'`, `'vue'`, …); none makes
  plain text (`.txt`). A leading dot and uppercase are fine. The doc keeps its first
  version's extension for good: a new version (`slug`) naming no extension keeps the
  doc's, and one naming a different extension is refused (`extension_mismatch`,
  "extension is fixed to csv for this doc"), as is an extension another doc type owns
  (`.html`, `.pdf`). `edit_text` keeps the doc's extension, and its copy mode copies
  it.
- **`list_documents`, `read_document` and the create tools' `metadata` carry the
  file's `extension`** beside `language`, both read off the stored file.

### Deprecated

- **`create_text`'s `type`, `language` and `filename` arguments.** They are no longer
  in its input schema but are still accepted: `type: 'md'` makes a Markdown doc, and a
  language token or a file name names the extension (`python` → `py`, `data.csv` →
  `csv`, `text` → `txt`). `extension` wins when both are given.

## [4.1.1] — 2026-10-20

### Changed

- **A staged draft's link is `reviewUrl` (was `shareUrl`), and it is described as
  private.** The create and edit tools return `reviewUrl`, the clean
  `app.docstash.ai/<slug>` link for the user to review the draft. Its description and
  the `delivery` note say it opens only for the user, their workspace members and
  people they have granted access, and tell the agent not to call it shareable or
  public. The stash note and the server instructions say the same of a saved doc.
- **Every create tool says a stashed doc is private until published.** `create_pdf`,
  `create_page` and `create_text` say a stashed doc is at a private url (the user,
  their workspace members, and people granted access) until published with
  `set_public_link`. `create_page` and the server instructions used to call a page's
  url "shareable".

### Fixed

- **PDF previews never inline the document's bytes on the tool result.** The inline
  preview renders a PDF only from its baked file, which it loads via `get_file`, so an
  inlined copy of the print-HTML source skipped that fetch and failed to render. No
  change to any tool's inputs.

## [4.1.0] — 2026-10-15

### Added

- **`read_document` reads multi-page sites.** An older web page made of several files
  (carried over from the retired app docs) has no single body, so `read_document`
  returns its files in a new `files` field (each file's path, size and text) instead
  of `content`. Read-only: `edit_page` can't change one; rebuild it as one page with
  `create_page`. A very large site returns its file paths only.

## [4.0.0] — 2026-10-15

### Removed (breaking)

- **`create_app` and `edit_app` are retired.** DocStash no longer hosts multi-file app
  bundles: a web app is one self-contained HTML file, made with `create_page` and
  changed with `edit_page`. The tools are gone from the tool list; a client calling one
  from a cached list gets an error pointing to `create_page` and `edit_page`.
- **`read_document` no longer takes `file` or returns `files`.** Those read one file of
  a multi-file app.

### Changed

- **`create_page` covers web apps and games.** Its description and the server
  instructions route live apps (React, three.js, d3, … loaded from a CDN) to it, as one
  HTML file.

## [3.0.0] — 2026-10-13

### Removed (breaking)

- **`create_docx`, `create_sheet`, `edit_docx` and `edit_sheet` are retired.** DocStash
  no longer creates or edits Word or spreadsheet documents. The tools are gone from the
  tool list; a client calling one from a cached list gets an error pointing to
  `create_pdf`, `create_text` and `create_page`.
- **Word and spreadsheet docs are view-only.** `read_document`, `show_docx` and
  `show_sheet` still work on existing ones, but no tool can add a version to them,
  restore an older version or commit a staged one. The server instructions no longer
  route any request to Word or spreadsheet output.

## [2.0.1] — 2026-10-10

### Changed

- **`openWorldHint` is now `true` on every tool that renders a document:** the `create_*`
  and `edit_*` tools, the `show_*` tools, `screenshot_document` and `stash`. Rendering
  loads the document's own fonts, images and scripts from whatever third-party hosts
  it references. No change to any tool's name, arguments or behavior.

## [2.0.0] — 2026-10-09

### Changed (breaking)

- **`manage_document` and `manage_sharing` are split into one tool per operation.** Each
  tool now takes only the inputs its operation needs and carries annotations that match
  what it does, so hosts can tell a read from a destructive change.
  - `manage_document` → `rename_document`, `set_trashed` (trash / restore; destructive),
    `revert_document`.
  - `manage_sharing` → `list_access` (read-only), `set_access` (Viewer / Editor / none;
    destructive), `set_public_link` (publish / unpublish; destructive).
  - The old names are no longer listed. Calls to them still work for now and run as the
    matching new tool, so clients holding an older tool list don't break. Each new tool
    returns the result shape the matching action did; `set_trashed` also reports `slug`
    and `trashed` in both directions.
  - `set_access` sets a level rather than adding or removing a grant: asking for the
    level a person already has, or removing access from someone with no grant, now
    succeeds without changing anything instead of returning an error.

- **The contract now lists every advertised tool,** including `get_document` and
  `get_file`, which only the inline preview calls. Each tool's `ui.visibility` is
  shown, so the two preview tools are marked as hidden from the agent.

- **The `instructions` and the `create_app` / `create_page` descriptions no longer
  tell the agent to prefer DocStash over other tools.** They now say what each tool
  takes and does: the authoring tools take source (a JSON spec for Word / Excel,
  HTML for a PDF), DocStash hosts pages and apps at a shareable URL, and
  `read_document` is how to get a DocStash link's content.

### Removed

- **`get_doc`**, the old name of the widget's internal `get_document` status call. No
  widget has called it since the rename, so it's no longer listed or accepted.

## [1.1.4] — 2026-09-28

### Changed

- **`create_sheet` / `edit_sheet`: the SheetSpec now matches exactly what renders — in
  BOTH the live preview and the downloaded .xlsx.** Everything the spec advertises is
  wired end-to-end through the grid engine and the .xlsx writer:
  - **Added / made real:** merged cells (`merges`), cell borders (`true` shortcut or
    per-side `{style, color}`), strikethrough, font family, row heights, data validation
    (dropdown lists + numeric/date bounds), autoFilter, and a wider set of number-format
    categories (percent / currency $€¥ / date / time / datetime / plain number). Columns
    also auto-fit to their content when no width is given.
  - **Removed:** `charts` and conditional formatting. Both were silently dropped when
    building the .xlsx (charts also only ever showed a placeholder in the preview) and
    are beyond the grid engine, so the spec no longer promises them.

## [1.1.3] — 2026-09-28

### Fixed

- **Corrected several inaccurate statements in the `instructions` and tool
  descriptions** — wording/accuracy only, no change to any tool's name or arguments:
  - The "show a doc to the user" pointers now name the real `show_*` tools
    (`show_pdf` / `show_docx` / `show_sheet` / `show_page` / `show_text`), not a
    non-existent `get_*`.
  - Fixed the "matching create create tool" wording in the iteration-vs-new-doc rule.
  - The slug-reuse warning is now accurate: reusing a **different-type** doc's slug is
    rejected with `format_locked` ("create a new document instead"), and reusing a
    **same-type** slug pushes a new version onto that existing doc — neither
    "overwrites" the other doc.
  - `discard` no longer claims "nothing was persisted"; it correctly states the unsaved
    draft is moved to **Trash** (recoverable, can't be stashed unless restored).
  - The app runtimes are described honestly: only `static` (HTML+CSS+JS, incl. the CDN
    frameworks) actually runs in the browser; other `language` tokens
    (`python` / `ruby` / `php` / …) are accepted but served as **source text**, not
    executed. Removed the misleading `app.py` runnable-entry examples.

## [1.1.2] — 2026-09-23

### Changed

- **`create_pdf` — the freeflow break primitives are now documented as
  ENGINE-RESERVED.** The description now states that `pdf-break` / `pdf-canvas` /
  `pdf-keep` / `pdf-bleed` must be applied as bare classes and never redefined or
  restyled (especially their `break-*` properties) — the engine owns those, and
  overriding them corrupts pagination (blank pages, spillover). Style the content
  INSIDE a primitive freely. Wording/clarity only; no change to any tool's
  arguments. (The bake now enforces this — reserved-primitive rules are locked at
  render time — and the render lint reports a `redefined-primitive` finding when an
  author overrides one.

## [1.1.1] — 2026-09-21

### Changed

- **The PDF preview widget now renders the REAL baked binary.** Previously it
  rendered the agent's print-HTML draft in a sandboxed shell. It now fetches the
  doc's baked PDF binary DIRECTLY from object storage over a short-lived signed
  URL (the storage origin declared in the widget CSP `connectDomains` in 1.1.0)
  and draws true paginated pages — real page breaks, backgrounds, full-bleed — on
  a canvas via pdf.js, identical to the app/public viewer and the downloaded
  file. When the binary can't be fetched (Claude iOS ignores the widget CSP, or
  none is baked yet) the widget shows an "Open in DocStash" message instead of the
  old draft. No change to how a pdf is authored or to any tool's arguments.

## [1.1.0] — 2026-09-21

### Changed

- **The widget now declares a Content-Security-Policy** so live html artifacts can
  load external resources instead of being pinned to inlined bytes. Previously the
  widget declared no CSP; external fonts and images were simply blocked. It now
  declares an allowlist, split by purpose (plus ChatGPT's legacy `openai/widgetCSP`):
  - `resourceDomains` (passive img/font/style/script) is broad, so LIVE html
    artifacts can load their assets at render time: the sanctioned CDNs authors are
    limited to (`fonts.googleapis.com`, `fonts.gstatic.com`, `cdnjs.cloudflare.com`,
    `cdn.jsdelivr.net`) plus an `https://*` wildcard for arbitrary image hosts where
    the host honors wildcard CSP.
  - `connectDomains` (fetch/XHR — the exfiltration surface) stays TIGHT and EXACT:
    only DocStash's own API origin plus the one object-storage origin (default store
    `SUPABASE_URL`). No wildcard — a broad connect-src would let a doc's JS POST data
    anywhere.
  The CSP is ADDITIVE over the host sandbox baseline and best-effort: Claude honors
  it on web only (iOS ignores all CSP — anthropics/claude-ai-mcp#40) and ChatGPT
  keys off `openai/widgetCSP`, so blocked resources are still surfaced honestly
  rather than depended on.

## [0.8.0] — 2026-09-21

### Added

- **`screenshot_document` gains two cost controls.** `resolution` sets the width
  in px of each returned page image (default 500 — enough to judge layout,
  overflow, colour and composition, and far cheaper on tokens; raise to ~1000–1600
  only to read fine print). `pages` takes an explicit 1-based set like `[2,5]` so,
  after editing specific pages, the untouched ones never enter context.
  `firstPage`/`lastPage` still select a contiguous range. Both size only the review
  images the model sees — never the baked PDF, which stays full fidelity.

## [0.7.0] — 2026-09-21

### Changed

- **BREAKING — `create_pdf` / `edit_pdf` now take ONE continuous, freeflow HTML
  document instead of hand-paginated page boxes.** You author semantic content
  only (headings, paragraphs, tables, images, lists) with no page boxes and no
  page-height math; DocStash's print engine paginates the flow automatically onto
  US Letter (816×1056px, ~682px content width per page) at bake time. The
  `content` argument is unchanged in name and type, but a doc built for the old
  fixed-page model must be reauthored as a freeflow document, and flow content must
  NEVER pin a height to the page (`100vh`, fixed sheets) — the engine owns
  pagination.
- **A PDF is a STATIC print document — no `<script>` / JS.** Any `<script>` is
  rejected at create/edit time (JS never runs at bake). Draw every chart, diagram,
  graphic, and signature motif as hand-authored inline SVG (+ CSS) — the contract
  now directs the agent to visualize (bar / line / donut, timelines, stat panels,
  iconography) this way rather than reaching for a scripting library.
- **PDF page images + page count are now rendered from the baked binary**, not a
  screen-media approximation, so `screenshot_document` and the reported page count
  reflect true pagination (short pages, real `@page` margins, full-bleed).
- **The engine trims a trailing all-blank page** a stray break can leave on the
  tail of a document.

### Added

- **`pdf-canvas` — a FIXED one-page design surface.** `<section
  class="pdf-canvas">` is exactly one sheet, edge-to-edge, and a positioning
  context: absolutely-position art, full-page backgrounds, and edge-anchored
  elements against KNOWN bounds, and compose it like a poster. Its content auto-fits
  to one page at bake, so the agent designs ~a page without measuring height. This
  is the primitive for the set-pieces that make a doc look designed — the cover,
  section dividers, and full-page infographics / hero spreads — alongside the
  flowing body.
- **Opt-in pagination primitives.** An element with class `pdf-break` forces a new
  page; `pdf-keep` keeps any card, stat box, figure, or callout whole (the print
  engine never splits a `pdf-keep` box across a page boundary — a box that does not
  fit the space left moves to the next page, the hook for the agent's own styled
  `<div>` blocks, since real `<table>`/`<tr>`/`<figure>`/`<img>` are already kept
  whole); and `<section class="pdf-bleed">` is a full-bleed page that still FLOWS —
  it starts a new page, paints its background to all four edges, and fills the
  sheet, but content longer than a page continues onto the next. The cover is just
  the first `pdf-bleed` (or a `pdf-canvas`).
- **Running header / footer via CSS `@page` margin boxes.** Put page numbers, a
  title strip, or a small logo on every page in any edge or corner, filled with the
  page counters (`counter(page)`, `counter(page) " / " counter(pages)`), a running
  `string-set` section title, or a `position:running()` element. Cover, `pdf-canvas`,
  and full-bleed pages are kept clear of it automatically.
- **Two render checks on `create_pdf` / `edit_pdf` `errors`:** `oversized-block`
  (a keep-whole box — `pdf-keep` / figure / image — taller than one page, so it
  can't stay whole) and `blank-interior-page` (an empty page mid-document from a
  stray break or an oversized element). Both are advisory warnings that never block
  a stash. The render-time `errors` set otherwise narrows to the freeflow signals
  (content too wide to paginate, a broken image, a blank document).

## [0.6.0] — 2026-09-16

### Changed

- **BREAKING — the per-type SHOW tools are renamed `get_*` → `show_*`.**
  `get_pdf` / `get_docx` / `get_sheet` / `get_page` / `get_text` are now
  `show_pdf` / `show_docx` / `show_sheet` / `show_page` / `show_text`. The name
  now matches what they do — DISPLAY a doc inline, off your context — and their
  "Show X" title, completing the `create_X` / `edit_X` / `show_X` triad per type
  and disambiguating them from `read_document` (which loads a doc's text INTO
  your context). Inputs and results are unchanged; only the names.

### Fixed

- **Show tools no longer duplicate the document into the model-visible result
  channel for hosts that fetch their own preview bytes** — the same gate 0.5.3
  added for create/edit now also covers the show tools. The source bytes ride
  the app-only fetch channel (kept out of the model's context); the inline
  fallback copy on the result is sent only to hosts that can't fetch it
  themselves. No change to the rendered preview.

## [0.5.4] — 2026-09-16

### Changed

- **Tool display titles unified across the create / edit / show families (no
  behavior change).** Each doc type's title now comes from one source, so it
  reads the same everywhere instead of drifting per family: "Edit Word doc" /
  "Show Word doc" (were "…Word (.docx) doc"), "Edit text or code" / "Show text
  or code" (were "…markdown / text / code doc"), and "Edit web app" (was "Edit
  multi-file app"). Tool names, inputs, and results are unchanged.

## [0.5.3] — 2026-09-16

### Fixed

- **Create/edit no longer duplicate the whole document into the model-visible
  result channel on hosts that fetch their own preview bytes.** The inline
  preview pulls a doc's source bytes over the app-only `get_file` channel, which
  keeps them out of the model's context. Create/edit ALSO inlined those bytes on
  the tool result as a fallback for hosts that can't make that fetch — but some
  hosts surface result content to their model, so a large document re-entered the
  model's context on every create/edit (and every surgical edit). The inline copy
  is now sent only to hosts that actually need it; hosts whose widget fetches its
  own bytes receive the identity handle only, and re-fetch the bytes off-context
  via `get_file`. No change to any tool's inputs or the rendered preview.

## [0.5.2] — 2026-09-16

### Fixed

- **`create_app` / `edit_app` no longer mount a perma-loading preview.** A
  multi-file app bundle has no in-widget renderer, so the inline preview widget
  mounted but never received bytes and loaded forever. Both tools now return the
  review-link surface on every host: the clean `app.docstash.ai/<slug>` link the
  user opens, with no widget mounted and no app source bytes riding the tool
  result. Every other type keeps its inline preview unchanged.

## [0.5.1] — 2026-09-13

### Changed

- **Tool surface de-duplicated (no behavior change).** Rules that were restated
  across the server instructions, tool descriptions, and result notes are now
  single-sourced (kept in one place with terse pointers elsewhere), shrinking the
  tool manifest. The agent-facing contract is unchanged.

## [0.5.0] — 2026-09-12

### Added

- **Surgical edits for the xlsx spec — `edit_sheet`.** The SheetSpec now edits in
  place like the docx spec: `{ old_string → new_string }` on the VISIBLE cell
  value (a number, a label, a hex color), so a large workbook no longer re-emits in
  full for a one-cell fix. DocStash JSON-escapes the strings — a quote or backslash
  in the edit can't break the stored spec — and re-validates the result against the
  schema, so a bad edit fails cleanly. Structural ADDITIONS are surgical too, via
  append ops that work on the parsed spec (so the escape-heal can't neutralize
  them): `addSheet` (a whole new sheet) / `addRows` (rows onto an existing sheet).
  The surgical `edit_*` family is now `edit_pdf` / `edit_docx` / `edit_sheet` /
  `edit_page` / `edit_text` / `edit_app`.

## [0.4.0] — 2026-09-12

### Changed

- **BREAKING — `create_docx` now takes a `DocxSpec` JSON, not Markdown.**
  `content` is `JSON.stringify(DocxSpec)`; the browser mints a real, editable
  .docx from it at view / download time. A Markdown body is no longer accepted.
  This mirrors `create_sheet` (agent emits a spec, not a binary).

### Added

- **Full Word fidelity on `create_docx` (the DocxSpec).** Document defaults
  (font / size / color / line spacing), named paragraph styles, page setup
  (size / orientation / margins / columns), headers & footers, a generated,
  clickable table of contents, six heading levels, per-run fonts / colors /
  sizes / links / super- and subscript / highlight, nested ordered and unordered
  lists, quotes, code blocks, dividers, page breaks, and embedded images.
- **Rich tables.** Column widths, cell merges (`colSpan` / `rowSpan`), per-cell
  shading, horizontal and vertical alignment (`valign`), `cellPadding`,
  `borderColor` / `borderWidth`, `banded` zebra rows, and table `align` on the
  page.
- **Surgical edits for the docx spec — `edit_docx`.** The DocxSpec now edits in
  place like every other type: `{ old_string → new_string }` on the VISIBLE text
  or value (a run, a heading, a hex color), so a 10-page doc no longer re-emits in
  full for a one-line fix. DocStash JSON-escapes the strings — a quote or backslash
  in the edit can't break the stored spec — and re-validates the result against the
  schema, so a bad edit fails cleanly. Structural ADDITIONS are surgical too, via
  an append op that works on the parsed spec (so the escape-heal can't neutralize
  it): `edit_docx` takes `appendBlocks` (+ an optional `after` text anchor). Only
  reorders, removals, or a near-total rewrite re-emit via `create_docx`. The
  surgical `edit_*` family is now `edit_pdf` / `edit_docx` / `edit_page` /
  `edit_text` / `edit_app`.

## [0.3.2] — 2026-09-05

### Added

- **`RESULTS.md` — the results half of the contract.** The mirror now documents
  what each tool hands BACK to the agent, per scenario (a clean document, layout
  errors, warnings, copy/template mode, screenshot QA, reads), alongside the tool
  descriptions and server instructions. No behavior change — it records the
  situational guidance the request handlers already return, so a reader gets the
  full contract, not just the call surface.

## [0.3.1] — 2026-09-05

### Fixed

- **Iteration now steers to the `edit_*` tools, not `create_*`.** The
  `screenshot_document` result and the create tools' "to revise this doc"
  guidance still pointed at the `create_*` tools on some paths, so an agent
  (ChatGPT most visibly) would re-author the whole document — or spawn a new
  one — instead of applying a surgical edit. Both now steer to the matching
  `edit_*` tool on the same slug; a type with no edit tool (`xlsx`) steers to
  `create_sheet`. Result-note wording only — the tool schemas and server
  `instructions` are unchanged from 0.3.0.

## [0.3.0] — 2026-09-04

### Added

- **Surgical edit tools — `edit_pdf` / `edit_docx` / `edit_page` / `edit_text` /
  `edit_app`.** The default way to change an existing doc: emit ONLY the deltas as
  `{ old_string → new_string }` pairs instead of re-authoring the whole document —
  far cheaper and it can't drift unrelated parts. One tool per type (mirroring the
  create_/get_ families) so each edit shows a FRESH inline preview of the new
  version. The create tools' guidance now points to them for every fix/tweak and
  reserves a full re-emit for near-total rewrites. Edits apply in order, match
  exactly (whitespace-tolerant) and must be unique (or `replaceAll`); a
  stale/ambiguous `old_string` fails cleanly without changing anything.
  - **In place** (default): the change becomes a new version of the doc.
  - **Copy mode** (`copy: true` + `name`): leaves the source untouched and creates a
    NEW doc — a filled-in copy with the edits applied. The template flow: keep one
    master, spin off variants (an invoice/offer-letter/report template → a copy per
    client).
  - `edit_app` edits a multi-file bundle by naming the `file` per edit. Every edit's
    raw `old_string`/`new_string` (and, for a copy, the source slug) is recorded on
    the version's history event. `xlsx` has no edit tool — re-emit with `create_sheet`.
- **`read_document` — multi-file app support.** A bare read of an `app` returns the
  bundle's file paths in `files`; passing `file` returns that one file's contents in
  `content` — so an agent can read a file, then change it with `edit_document`.

## [0.2.3] — 2026-09-03

### Added

- **`create_pdf` — automatic page margins.** Body content that would otherwise run
  flush to the sheet edge now gets a comfortable margin at render time, and DocStash
  honors padding set on `.ds-page` (a page split into regions previously discarded it,
  printing content flush to the paper edge). Keep authoring your own margins as before;
  add `class="ds-page ds-bleed"` to opt a page out for a full-bleed cover, background,
  or edge-to-edge image. The tool description documents this.
- **`box-overflow` — new PDF render finding.** The `errors` report now flags content
  wider than its own box or table cell (a long unbreakable string, a `nowrap` line, an
  oversized child, a table wider than its column), which spills or clips on a fixed
  page. Advisory, like every other finding — nothing blocks a stash.

### Fixed

- **ChatGPT inline-preview widget.** Restored the `openai/outputTemplate` `_meta`
  binding on the widget tool definitions and their results. Without it ChatGPT mounted
  the preview widget but never received the tool result, so the widget loaded
  indefinitely; it renders again now. No effect on other hosts, which bind the widget
  from the tool's static `_meta` and ignore the key.

## [0.2.2] — 2026-09-02

### Changed

- **`manage_sharing` display title corrected.** Its human-readable title (shown in
  clients that render one) is now "Manage sharing" — sentence case, matching every other
  tool — instead of the raw `manage_sharing` id it had been left as in 0.2.1. The callable
  tool name is unchanged, so this is display-only and nothing breaks for integrators.

## [0.2.1] — 2026-09-01

### Fixed

- **`manage_sharing` action='add' return value.** Granting a viewer or editor read a
  singular `grant` field the API does not return — the API responds with the full,
  refreshed `grants` list — so the tool built a malformed result (and could throw) even
  though the grant itself succeeded. It now resolves the just-added grant from that list by
  email and returns the correct `{ grantId, slug, email, level, url }` (`grantId` is null
  if the row cannot be matched, but the grant still applies).

## [0.2.0] — 2026-09-01

### Added

- **`create_pdf` — page regions.** A PDF page can now declare semantic bands: `.ds-header`
  and `.ds-footer` (pinned running chrome), `.ds-content` (the body), and `.ds-bg` (a
  full-bleed background). A header/footer declared once at the top level repeats on every
  page except the cover; `.ds-pageno` / `.ds-pagetotal` spans are filled automatically
  (no more hand-numbering); and the body is bounded to its region, so an overrun shrinks
  the content (a `squeezed` warning) instead of colliding with the chrome. A plain
  `.ds-page` with content directly inside still renders exactly as before — regions are
  opt-in and additive.

### Changed

- **`create_pdf` page contract rewritten regions-first.** Shrink-to-fit is now described
  once (content-scoped), with the legacy whole-page shrink framed as the bare-page fallback.
- **Contract streamlined (~37% smaller tool manifest, no behavioral change).** The
  author→review→stash lifecycle, the slug/versioning policy, the attribution rule, the
  image-URL + font recipe, the responsive-HTML rule, and the `read_document` / `get_*`
  SHOW-vs-READ guidance are now stated once in the server `instructions` and only pointed
  to from each tool, instead of being repeated verbatim in every one. Each tool description
  now carries just its own spec plus a short pointer — which also lowers the chance a
  long-chat client truncates the tool list.

## [0.1.0] — 2026-08-25

- Initial public contract: the authoring tools (`create_pdf` / `create_docx` /
  `create_sheet` / `create_app` / `create_page` / `create_text`), the per-type show tools,
  the reads (`list_documents` / `read_document` / `list_org_members` /
  `screenshot_document`), the mutations (`manage_document` / `manage_sharing`), `stash` /
  `discard`, and organization targeting (`get_organizations` / `set_organization`) — see
  [`TOOLS.md`](./TOOLS.md) and [`INSTRUCTIONS.md`](./INSTRUCTIONS.md).
