# DocStash MCP — public contract

A read-only mirror of what the DocStash MCP server tells a connected agent:
the always-on server instructions and every tool the agent can call, with the
exact descriptions and input schemas it receives.

It is published for transparency. It is not a library and is not meant to be
built — it is generated from the DocStash source and mirrored here as text.

- [`INSTRUCTIONS.md`](./INSTRUCTIONS.md) — the server instructions.
- [`TOOLS.md`](./TOOLS.md) — every advertised tool: description + input schema.
- [`tools.json`](./tools.json) — the same tools, machine-readable.
- [`RESULTS.md`](./RESULTS.md) — what each tool returns to the agent, per scenario.
- [`CHANGELOG.md`](./CHANGELOG.md) — version history of the contract.

Two tools, `get_document` and `get_file`, are called only by the inline preview
shown in the chat: it uses them to check a document's status and load its
contents. They're marked `ui.visibility: ["app"]`, so hosts hide them from the
agent, but they're listed here because `tools/list` advertises them.
