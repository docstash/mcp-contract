<!-- Hand-maintained. Not generated. Keep in step with the server's result notes by hand. -->
# DocStash MCP — tool results

What each tool hands BACK to the agent, per scenario. `TOOLS.md` and
`INSTRUCTIONS.md` cover what the agent reads BEFORE acting (descriptions, input
schemas, the always-on instructions); this covers what it reads AFTER — the
payload and the situational note that steer the fix/iterate loop.

Every response is a small JSON payload plus a plain-text note written to the
agent. Below, the payload is shown as example JSON and the note is reproduced
verbatim; `<slug>` and other interpolated values are filled with examples.

---

## `create_pdf` / `create_page` / `create_text` and `edit_pdf` / `edit_page` / `edit_text`

A create or edit STAGES a new version (it does not save or publish). Payload
(FULL palette):

```json
{
  "preview": { "slug": "k3n9qz", "type": "pdf", "name": "Q3 Report", "artifactId": "a1b2c3d4" },
  "url": "https://app.docstash.ai/k3n9qz?v=a1b2c3d4",
  "reviewUrl": "https://app.docstash.ai/k3n9qz",
  "delivery": "Give the user `reviewUrl` as a plain https link to review the draft. It is PRIVATE (opens only for the user, their workspace members, and people they have granted access): do NOT call it shareable, public, or a link to send to others. The inline preview is the deliverable. Do NOT generate, attach, or offer a downloadable file (no .pdf binary, no /mnt/data path)."
}
```

When the push-time lint found anything, the payload adds `lint` (see
[Lint findings](#lint-findings) below). No `lint` = clean.

On a host that renders the inline preview, `structuredContent` also carries the
preview's identity (`slug`, `type`, `name`, `metadata`, `artifactId`, `url`).

### LITE palette

No inline preview: the user reviews the draft at the link. Payload:

```json
{
  "unsavedVersion": { "slug": "k3n9qz", "type": "html", "name": "Budget Tracker", "artifactId": "a1b2c3d4" },
  "url": "https://app.docstash.ai/k3n9qz?v=a1b2c3d4",
  "reviewUrl": "https://app.docstash.ai/k3n9qz",
  "delivery": "This is an UNSAVED DRAFT — NOT saved, stashed, or public. Do NOT tell the user you saved/stashed/published it; describe it as a draft/preview only. Give the user `reviewUrl` as a plain https link to review it, and tell them it is a draft they must review and Stash to keep. The link is PRIVATE (only the user, their workspace members, and people they have granted access can open it): do NOT call it shareable or public. No downloadable file (.pdf, no /mnt/data)."
}
```

When the lint found anything, the payload adds `lint`, as on the FULL palette.

`say`: `Draft staged — NOT saved or stashed yet; the user must Stash it to keep it.`

`agentNote`:

> UNSAVED draft staged — tell the user plainly it is a draft they must review and Stash to keep; do NOT call stash yourself. To revise this doc, call `edit_page` with slug=k3n9qz (type=html) (emit ONLY the changed spans as edits, not the whole doc) — do NOT omit slug, that creates a different new doc.

### Clean document (no lint findings, inline preview)

`say`: `Preview ready — review it before stashing.`

`agentNote`:

> Preview shown; NOT stashed — present it and STOP, and do NOT call stash unless the user explicitly asks. To revise this doc, call `edit_pdf` with slug=k3n9qz (type=pdf) (emit ONLY the changed spans as edits, not the whole doc) — do NOT omit slug, that creates a different new doc. To throw it away, call discard with slug=k3n9qz.

(The "no binary / give the user the private `reviewUrl` as a plain link" rule rides the `delivery` structuredContent field above — the channel a content-dropping host like ChatGPT reads — and the stash-only-when-asked rule is on every create tool via the shared lifecycle prose + the server instructions, so the agentNote doesn't restate either.)

### Lint findings

When the push-time lint found anything, the payload carries `lint`, the `say`
becomes `Preview has lint findings (<verdict>).`, and the `agentNote` opens with
`Lint: <verdict> (see \`lint\`).` before the surface's own note. A PDF:

```json
"lint": {
  "verdict": "2 errors, 2 warnings",
  "findings": [
    { "severity": "error",
      "source": "<img class=\"cover-image\" src=\"https://upload.wikimedia.org/wikipedia/commons/2/2b/Hugh_Laurie_2012.jpg\">",
      "issue": "image returned HTTP 404" },
    { "severity": "error",
      "source": "<img class=\"portrait\" src=\"https://images.example.com/team.jpg\">",
      "issue": "image request failed: DNS lookup failed" },
    { "severity": "warning",
      "source": "<table class=\"data-table\">",
      "issue": "84px wider than the page" },
    { "severity": "warning",
      "issue": "page 4 is blank" }
  ],
  "next": "Edit with edit_pdf using `source` as old_string. Errors lose content; warnings may be intentional design. screenshot_document shows pages."
}
```

A web page that crashed:

```json
"lint": {
  "verdict": "1 error",
  "findings": [
    { "severity": "error", "issue": "uncaught exception: ReferenceError: useState is not defined" }
  ],
  "next": "Edit with edit_page using `source` as old_string. Errors lose content; warnings may be intentional design. screenshot_document shows the page at desktop and mobile."
}
```

Each finding is ONE occurrence (each broken `<img>`, each repeat, each element
sitting on other text); only exact duplicates collapse. `source` is the
offending text exactly as written, widened until it occurs once in the doc, and
is absent when there's nothing to quote. `issue` is a fixed sentence of what
was measured, with no advice. The full set:

| Finding | `issue` |
|---|---|
| broken image | `image returned HTTP 404` · `image request failed: DNS lookup failed` (connection refused, timed out, connection reset, TLS certificate error, blocked by the browser) · `image URL returned text/html` · `image data could not be decoded` · `img has no src` |
| repeated image | `same image URL as <the first slot's source>` |
| too wide (PDF) | `700px wider than the page` |
| too wide (page) | `214px wider than the 390px mobile viewport` (or `page is … wider …` when no one element accounts for it) |
| too tall (PDF) | `1404px tall; a page holds 922px` |
| overlapping text | `overlaps <the covered elements' source>` |
| restyled engine class (PDF) | `CSS restyles .pdf-canvas; the PDF engine overrides this class` (source: the CSS rule) |
| blank page (PDF) | `page 4 is blank` |
| empty document | `document has no text or media` |
| blank render (page) | `page painted nothing` |
| no viewport tag | `no <meta name="viewport">` |
| missing file | `https://…/app.js returned HTTP 404` (source: the tag that references it) |
| crash | `uncaught exception: <message>` |
| console error | `console error: <message>` |

An element the source can't account for (one a script built at runtime) has no
`source`; its issue names it by its text: `element not found in the source
("Team photo"): image returned HTTP 404`.

### Copy / template mode (`edit_*` with `copy: true` + `name`)

The edit lands on a NEW document (a filled copy); the source is untouched. The
returned `preview` points at the new copy's slug, and its version history records
the source slug plus each edit's raw `old_string`/`new_string`.

---

## `screenshot_document`

Renders the staged working copy for the agent's own QA. Payload plus the page
images as JPEG image content blocks (PDF: one per page, up to 8 per call; every
other type: one bounded full-height shot at desktop and at mobile):

```json
{
  "slug": "k3n9qz",
  "type": "pdf"
}
```

When the images cover only part of a PDF, the payload adds
`"pageRange": { "from": 1, "to": 8, "total": 20 }` and `"pages": [1, 2, 3, 4, 5, 6, 7, 8]`.

`lint` (when there are findings) is the push-time lint, reused (the artifact is
immutable, so the geometry cannot have changed).

### Findings present

> Lint: 2 errors, 2 warnings (see `lint`). Fix them by EDITING this same doc. To revise this doc, call `edit_pdf` with slug=k3n9qz (type=pdf) (emit ONLY the changed spans as edits, not the whole doc) — do NOT omit slug, that creates a different new doc.

When the images don't cover the whole doc, the note appends the page window, e.g.:

> The images below are pages 1–8 of 20. To see more pages, call screenshot_document again with firstPage/lastPage — or pages=[…] for a specific set — (max 8 per call), e.g. firstPage=9, lastPage=16.

(An explicit `pages` selection gets `The images below are page(s) 2, 5 of 20.` with no "see more" nudge.)

### No findings

> No lint findings in k3n9qz. Review the 5 page image(s) below for anything geometry can't catch — text hidden behind a block, wrong colours, a broken visual layout.

With no images: `No lint findings in k3n9qz. (No page images available for this type.)`

---

## `read_document`

A silent read — nothing is rendered or shown to the user. Returns the doc's
metadata, a reference to its latest version, `content` (the body text for
text-like kinds — html/md/text, agent-authored pdf print-HTML, and the
xlsx/docx specs; binary uploads are not text-extractable), `organizationId` +
`sameOrganization`, and the latest version's `lint` when it has findings. With
`withHistory: true` it adds `history` (activity events, newest first; pushed
versions carry the `artifactId` that `revert_document` takes). This
is also how to re-check findings mid-fix — there is no separate lint tool. A multi-page site (an
older web page made of several files) has no `content`: its files come back in
`files` (`path`, `size`, `content` each), read-only, with a note to rebuild it as one
page via `create_page` to change it; a site over ~500 KB of text lists `path` and
`size` only.

`agentNote` opens with `SILENT read — nothing was rendered or shown to the user.`,
then appends only the clauses that apply:

- **content not inlinable** (binary upload): how to fetch it from the artifact's `fileUrl`.
- **cross-organization** (`sameOrganization: false`):
  > NOTE: this doc lives in a DIFFERENT workspace than your active one. Reading across workspaces is fine, but a NEW doc you author lands in the ACTIVE workspace: call get_organizations / set_organization first if it should go here instead.
- **lint findings**: `Lint on this version: <verdict> (see `lint`).`
- **slug steering**: the shared "revise with the matching `edit_*` tool on the same slug" line, plus a display hint: on FULL, call the matching `show_*` tool (or share the URL for a type with no inline preview); on LITE, share the URL. For a never-stashed doc it adds that the doc is UNSAVED and to call `stash` with the slug to publish it.

---

## `stash` / `discard` / the lifecycle and sharing tools / `list_documents` / `list_org_members` / `get_organizations` / `set_organization` / the `show_*` tools

- **`stash`** — saves the staged version (the doc stays PRIVATE, not published) and returns the doc's `url`. To make it public afterward, call `set_public_link` with `isPublic: true`. When the staged version already matches the live one, nothing is committed and the result carries `committed: false`.
- **`discard`** — on a doc with a live version, drops the staged working copy and keeps the live version; on a never-stashed draft, moves the whole draft to Trash (recoverable).
- **the `show_*` tools** (`show_pdf` / `show_page` / `show_text`) — render the doc inline for the user and echo the same slug-continuity guidance (revise with the matching `edit_*` tool on the same slug; Word and spreadsheet docs are view-only).
- **the lifecycle tools** (`rename_document` / `set_trashed` / `revert_document`), **the sharing tools** (`list_access` / `set_access` / `set_public_link`), **`list_documents`**, **`list_org_members`**, **`get_organizations`**, **`set_organization`** — return the shapes documented on each tool's own entry in `TOOLS.md`.
