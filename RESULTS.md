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

## `create_pdf` / `create_docx` / `create_sheet` / `create_page` / `create_text` / `create_app` and `edit_pdf` / `edit_docx` / `edit_page` / `edit_text` / `edit_app`

A create or edit STAGES a new version (it does not save or publish). Payload:

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
= real content loss, `"warning"` = advisory). Empty/absent = clean.

### Clean document (no errors)

`say`: `Preview ready — review it before stashing.`

`agentNote`:

> Preview shown; NOT stashed. A plain create/generate ask is now DONE — present the preview and STOP: call stash with slug=k3n9qz ONLY when the user EXPLICITLY says to save / stash / publish it (or clicks Stash in the inline preview). The inline preview IS the deliverable — never generate, attach, or hand back a file (no binary .pdf/.docx/.xlsx, no download card, no /mnt/data path); if you reference it in text, share `shareUrl` (https://app.docstash.ai/k3n9qz) as a plain https hyperlink and nothing else. To revise this doc, call `edit_pdf` with slug=k3n9qz (type=pdf) (emit ONLY the changed spans as edits, not the whole doc) — do NOT omit slug, that creates a different new doc. To throw it away, call discard with slug=k3n9qz.

### PDF with layout errors (`severity: "error"`)

`say`: `Preview has layout errors — fix them before stashing.`

`agentNote` (the `errorNote` prefixes the note above; here `errors` carries the findings):

> ⚠️ 2 LAYOUT ERROR(S) in your PDF — a reader would LOSE content (text past the page edge / colliding):
>   • page 3: text overflows the page bottom by 84px (.ds-content > .section:last-child)
>   • page 5: two blocks overlap (.chart over .caption)
> Nothing reflows — YOU own pagination: SPLIT content across more `.ds-page` blocks or recompose the page — do NOT strip design elements or shrink the font to force a fit.
> NEXT STEP: call `screenshot_document` with slug=k3n9qz to SEE the rendered pages before rewriting — pass firstPage/lastPage to screenshot ONLY the pages the findings name (not the whole doc); the images show exactly what broke, so you fix it in one pass instead of guessing from the numbers. Then apply the fix with `edit_pdf` (slug=k3n9qz) — emit just the changed spans, not the whole doc — and screenshot the same pages again until clean. (Re-run `create_pdf` with the same slug only for a near-total rewrite.)

### Web page / app that is broken (`severity: "error"`)

> ⚠️ Your app is BROKEN (crash / blank render / missing file):
>   • console error while loading: ReferenceError: useState is not defined
> NEXT STEP: call `screenshot_document` with slug=k3n9qz to SEE the rendered result (desktop + mobile), then apply the fix with `edit_app` (slug=k3n9qz) — just the changed spans — or re-run `create_app` with the same slug for a near-total rewrite.

### Warnings (`severity: "warning"`)

When findings are warnings (not errors), this block is appended instead of the
error block:

> Warnings (advisory, NOT verdicts — the linter reports geometry, it cannot see intent. Judge each with your own design eyes: an intentional full-bleed glyph, watermark, or decorative bleed that trips a check is FINE — leave it. Fix only what a reader would actually experience as broken, and NEVER strip design elements or shrink the font just to silence a warning):
>   • page 1: element bleeds 6px past the right edge (.hero-image)

### Copy / template mode (`edit_*` with `copy: true` + `name`)

The edit lands on a NEW document (a filled copy); the source is untouched. The
returned `preview` points at the new copy's slug, and its version history records
the source slug plus each edit's raw `old_string`/`new_string`.

---

## `screenshot_document`

Renders the staged working copy for the agent's own QA. Payload plus the page
images as image content blocks (PDF: one per page, up to 8 per call; every other
type: one bounded full-height shot at desktop and at mobile):

```json
{
  "slug": "k3n9qz",
  "type": "pdf",
  "errors": [],
  "fullHeight": ["<base64 jpeg>", "…"],
  "pageStart": 0,
  "pageCount": 5
}
```

`errors` is reused from the push-time lint (the artifact is immutable, so the
geometry cannot have changed).

### Errors present

> 2 layout error(s) in k3n9qz:
>   • page 3: text overflows the page bottom by 84px (.ds-content > .section:last-child)
>   • page 5: two blocks overlap (.chart over .caption)
> Fix them by EDITING this same doc — call `edit_pdf` with slug=k3n9qz and emit ONLY the changed spans. Do NOT create a new document (that spawns a duplicate); re-run `create_pdf` with the SAME slug only for a near-total rewrite.

For a spreadsheet (no edit tool), the fix tool is `create_sheet` on the same slug.

When the images don't cover the whole doc, the note appends the page window, e.g.:

> The images below are pages 1–8 of 20. To see more pages, call screenshot_document again with firstPage/lastPage (max 8 pages per call), e.g. firstPage=9, lastPage=16.

### No errors

> No deterministic layout errors in k3n9qz. Review the 5 page image(s) below for anything geometry can't catch — text hidden behind a block, wrong colours, a broken visual layout.

---

## `read_document`

A silent read — nothing is rendered or shown to the user. Returns the doc's
metadata, a reference to its latest version, `content` (the body text for
text-like kinds — html/md/text, agent-authored pdf print-HTML, and the
xlsx/docx specs; binary uploads are not text-extractable), and any unresolved
lint `errors` on the latest version. This is also how to re-check errors
mid-fix — there is no separate errors tool. For an `app`, a bare read returns
the bundle's file paths in `files`; passing `file` returns that one file's
contents in `content`.

---

## `stash` / `discard` / `manage_document` / `manage_sharing` / `list_documents` / `list_org_members` / `get_organizations` / `set_organization` / the `get_*` show tools

- **`stash`** — saves the staged version (the doc stays PRIVATE, not published) and returns the doc's `url`. To make it public afterward, call `manage_sharing` with `action: "set-public"`.
- **`discard`** — drops the staged working copy, keeping the live version.
- **the `get_*` show tools** (`get_pdf` / `get_docx` / `get_sheet` / `get_page` / `get_text`) — render the doc inline for the user and echo the same slug-continuity guidance (revise with the matching `edit_*` tool on the same slug).
- **`manage_document`** (rename / trash / restore / delete), **`manage_sharing`**, **`list_documents`**, **`list_org_members`**, **`get_organizations`**, **`set_organization`** — return the shapes documented on each tool's own entry in `TOOLS.md`.
