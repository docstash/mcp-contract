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
  "shareUrl": "https://app.docstash.ai/k3n9qz",
  "delivery": "Share `shareUrl` with the user as a plain https link — the inline preview is the deliverable. Do NOT generate, attach, or offer a downloadable file (no .pdf/.docx/.xlsx binary, no /mnt/data path).",
  "errors": []
}
```

`errors` is the push-time lint, each finding stamped with `severity` (`"error"`
= real content loss, `"warning"` = advisory). Empty = clean.

On a host that renders the inline preview, `structuredContent` also carries the
preview's identity (`slug`, `type`, `name`, `metadata`, `artifactId`, `url`).

### LITE palette

No inline preview: the user reviews the draft at the link. Payload:

```json
{
  "unsavedVersion": { "slug": "k3n9qz", "type": "html", "name": "Budget Tracker", "artifactId": "a1b2c3d4" },
  "url": "https://app.docstash.ai/k3n9qz?v=a1b2c3d4",
  "shareUrl": "https://app.docstash.ai/k3n9qz",
  "delivery": "This is an UNSAVED DRAFT — NOT saved, stashed, or public. Do NOT tell the user you saved/stashed/published it; describe it as a draft/preview only. Share `shareUrl` as a plain https link and tell them it is a draft they must review and Stash to keep. No downloadable file (.pdf/.docx/.xlsx, no /mnt/data).",
  "errors": []
}
```

When there are findings, a `feedback` field repeats the findings note below.

`say`: `Draft staged — NOT saved or stashed yet; the user must Stash it to keep it.`

`agentNote`:

> UNSAVED draft staged — tell the user plainly it is a draft they must review and Stash to keep; do NOT call stash yourself. To revise this doc, call `edit_page` with slug=k3n9qz (type=html) (emit ONLY the changed spans as edits, not the whole doc) — do NOT omit slug, that creates a different new doc.

### Clean document (no errors, inline preview)

`say`: `Preview ready — review it before stashing.`

`agentNote`:

> Preview shown; NOT stashed — present it and STOP, and do NOT call stash unless the user explicitly asks. To revise this doc, call `edit_pdf` with slug=k3n9qz (type=pdf) (emit ONLY the changed spans as edits, not the whole doc) — do NOT omit slug, that creates a different new doc. To throw it away, call discard with slug=k3n9qz.

(The "no binary / share `shareUrl` as a plain link" rule rides the `delivery` structuredContent field above — the channel a content-dropping host like ChatGPT reads — and the stash-only-when-asked rule is on every create tool via the shared lifecycle prose + the server instructions, so the agentNote doesn't restate either.)

### PDF with layout errors (`severity: "error"`)

`say`: `Preview has layout errors — fix them before stashing.` (used whenever `errors` is non-empty, warnings included)

`agentNote` (this findings block prefixes the clean-document note above; here `errors` carries the findings):

> ⚠️ 2 LAYOUT ERROR(S) in your PDF — content that can't render cleanly (too wide to paginate, a broken image, or a blank document):
>   • page 2 — img.logo is a broken image (loaded with no dimensions) — check the src is a DIRECT image URL that returns bytes
>   • page 5 — img.hero is a broken image (loaded with no dimensions) — check the src is a DIRECT image URL that returns bytes
> The engine paginates your freeflow document automatically — fix the offending content itself (let a wide element wrap, break an unbreakable string, repair a broken image) — do NOT strip design elements or shrink the font to force a fit.
> NEXT STEP: call `screenshot_document` with slug=k3n9qz to SEE the rendered pages before rewriting — pass firstPage/lastPage to screenshot ONLY the pages the findings name (not the whole doc); the images show exactly what broke, so you fix it in one pass instead of guessing from the numbers, then screenshot again until clean.

(The clean-document note — including the "revise with `edit_pdf` on the same slug, don't omit it" line — follows, so the fix/iterate rule isn't re-stated here.)

### Web page that is broken (`severity: "error"`)

> ⚠️ Your page is BROKEN (crash / blank render / missing file):
>   • uncaught exception while loading: ReferenceError: useState is not defined
> NEXT STEP: call `screenshot_document` with slug=k3n9qz to SEE the rendered result (desktop + mobile) and fix what's wrong.

(The surface's own note — here the LITE note with the `edit_page` steering line — follows.)

### Warnings (`severity: "warning"`)

Warning findings get this block, after the error block when both are present
(and before the NEXT STEP line):

> Warnings (advisory, NOT verdicts — the linter reports geometry, it cannot see intent. Judge each with your own design eyes: an intentional full-bleed glyph, watermark, or decorative bleed that trips a check is FINE — leave it. Fix only what a reader would actually experience as broken, and NEVER strip design elements or shrink the font just to silence a warning):
>   • page 3 — .data-table runs 84px wider than the page can hold — content that can't paginate to a width (an unbreakable URL/string, nowrap text, or a too-wide table). Let it wrap, break the string, or shrink/scroll the table so it fits the column

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
  "type": "pdf",
  "errors": []
}
```

When the images cover only part of a PDF, the payload adds
`"pageRange": { "from": 1, "to": 8, "total": 20 }` and `"pages": [1, 2, 3, 4, 5, 6, 7, 8]`.

`errors` is reused from the push-time lint (the artifact is immutable, so the
geometry cannot have changed).

### Errors present

> 2 layout error(s) in k3n9qz:
>   • page 2 — img.logo is a broken image (loaded with no dimensions) — check the src is a DIRECT image URL that returns bytes
>   • page 5 — img.hero is a broken image (loaded with no dimensions) — check the src is a DIRECT image URL that returns bytes
> Fix them by EDITING this same doc. To revise this doc, call `edit_pdf` with slug=k3n9qz (type=pdf) (emit ONLY the changed spans as edits, not the whole doc) — do NOT omit slug, that creates a different new doc.

When the images don't cover the whole doc, the note appends the page window, e.g.:

> The images below are pages 1–8 of 20. To see more pages, call screenshot_document again with firstPage/lastPage — or pages=[…] for a specific set — (max 8 per call), e.g. firstPage=9, lastPage=16.

(An explicit `pages` selection gets `The images below are page(s) 2, 5 of 20.` with no "see more" nudge.)

### No errors

> No deterministic layout errors in k3n9qz. Review the 5 page image(s) below for anything geometry can't catch — text hidden behind a block, wrong colours, a broken visual layout.

With no images: `No deterministic layout errors in k3n9qz. (No page images available for this type.)`

---

## `read_document`

A silent read — nothing is rendered or shown to the user. Returns the doc's
metadata, a reference to its latest version, `content` (the body text for
text-like kinds — html/md/text, agent-authored pdf print-HTML, and the
xlsx/docx specs; binary uploads are not text-extractable), `organizationId` +
`sameOrganization`, and any unresolved lint `errors` on the latest version. With
`withHistory: true` it adds `history` (activity events, newest first; pushed
versions carry the `artifactId` that `revert_document` takes). This
is also how to re-check errors mid-fix — there is no separate errors tool. A multi-page site (an
older web page made of several files) has no `content`: its files come back in
`files` (`path`, `size`, `content` each), read-only, with a note to rebuild it as one
page via `create_page` to change it; a site over ~500 KB of text lists `path` and
`size` only.

`agentNote` opens with `SILENT read — nothing was rendered or shown to the user.`,
then appends only the clauses that apply:

- **content not inlinable** (binary upload): how to fetch it from the artifact's `fileUrl`.
- **cross-organization** (`sameOrganization: false`):
  > NOTE: this doc lives in a DIFFERENT workspace than your active one. Reading across workspaces is fine, but a NEW doc you author lands in the ACTIVE workspace: call get_organizations / set_organization first if it should go here instead.
- **unresolved findings**: the findings split into errors and warnings (same per-finding lines as the authoring notes), plus a pointer to `screenshot_document` to see the rendered pages.
- **slug steering**: the shared "revise with the matching `edit_*` tool on the same slug" line, plus a display hint: on FULL, call the matching `show_*` tool (or share the URL for a type with no inline preview); on LITE, share the URL. For a never-stashed doc it adds that the doc is UNSAVED and to call `stash` with the slug to publish it.

---

## `stash` / `discard` / the lifecycle and sharing tools / `list_documents` / `list_org_members` / `get_organizations` / `set_organization` / the `show_*` tools

- **`stash`** — saves the staged version (the doc stays PRIVATE, not published) and returns the doc's `url`. To make it public afterward, call `set_public_link` with `isPublic: true`. When the staged version already matches the live one, nothing is committed and the result carries `committed: false`.
- **`discard`** — on a doc with a live version, drops the staged working copy and keeps the live version; on a never-stashed draft, moves the whole draft to Trash (recoverable).
- **the `show_*` tools** (`show_pdf` / `show_docx` / `show_sheet` / `show_page` / `show_text`) — render the doc inline for the user and echo the same slug-continuity guidance (revise with the matching `edit_*` tool on the same slug; Word and spreadsheet docs are view-only).
- **the lifecycle tools** (`rename_document` / `set_trashed` / `revert_document`), **the sharing tools** (`list_access` / `set_access` / `set_public_link`), **`list_documents`**, **`list_org_members`**, **`get_organizations`**, **`set_organization`** — return the shapes documented on each tool's own entry in `TOOLS.md`.
