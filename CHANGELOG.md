# Changelog

Changes to the DocStash MCP server's agent-facing contract — the tools (their
descriptions and input schemas) and the always-on server `instructions` — recorded
alongside the contract they describe. The current version is reported as
`serverInfo.version` in the MCP `initialize` handshake.

The format follows [Keep a Changelog](https://keepachangelog.com/), and versioning is
[semantic](https://semver.org/):

- **MAJOR** — a breaking change to a tool's name, its arguments, or a required input.
- **MINOR** — a new tool, or a new capability / expanded contract on an existing one.
- **PATCH** — wording, clarity, or de-duplication that does not change behavior. Patch
  bumps are applied automatically when the specs change; MINOR / MAJOR are made by hand.

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
