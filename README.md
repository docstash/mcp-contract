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

Widget-internal tools (the preview bridge) are omitted — they are plumbing the
model never calls, not part of the authoring contract.
