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
