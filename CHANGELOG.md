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
