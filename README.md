# @pipeworx/fr-rappelconso

RappelConso, France's official consumer product recall register run by the DGCCRF: food, baby and children's goods, electrical appliances, cosmetics, home goods, sports goods, clothing and vehicles (about 19,000 recalls), with the reason for the recall, the risks, the instructions to consumers, distributors, sale period and batch identifiers.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2007+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `fr_rappelconso_search(query?, category?, date_from?, date_to?, limit?)` — searches the recalls, newest first. The register is written in French, so French words match best (`salmonelle`, `fromage`, `poussette`); an English query that matches nothing says so. `category` takes the register's categories (`alimentation`, `bébés-enfants (hors alimentaire)`, `hygiène-beauté`, ...) or English words (food, vehicles, baby, cosmetics, home, sports, clothing). Dates filter on the publication date across the whole list. Each hit: id, reference, publication date, category and sub-category, product name, brand, reason and risks as stated, and the official notice link. `total_matching` counts the whole filtered list; `returned` is capped by `limit`. Zero matches states the list and filters searched.
- `fr_rappelconso_get(id)` — one recall by its id (such as `23963`) or reference (such as `2026-10-0288`): product identification (barcodes, batches, dates), packaging, sale period and area, storage, distributors, reason, risks, health recommendations and consumer instructions as stated, compensation, and links to the notice, poster and product list. An unknown id is `found:false`.

Every response states the list searched, the period covered and `data_as_of` (the newest publication date in the page).

## Auth

Keyless.

## Data sources

- DGCCRF (Direction générale de la concurrence, de la consommation et de la répression des fraudes, France), RappelConso: consumer product recalls. Read live per call, nothing stored. Licence: Licence Ouverte v2.0 (Etalab), which allows reuse with attribution to the publisher and the date of the data; each response carries `source`, `licence` and `data_as_of`.

Privacy: company and brand names are returned; the notice's contact phone number is dropped, and e-mail addresses and phone numbers in free text are removed.

Traps worth knowing: text is French and quoted as published; recalls of the same notice can exist in several versions (the latest is returned); barcodes and batch numbers sit in one mixed list (`product_identification`) in the order the publisher gives them; zero matches means no recall in this list, not that a product is safe.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "fr-rappelconso": {
      "url": "https://gateway.pipeworx.io/fr-rappelconso/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/fr-rappelconso/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 2007+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/fr_rappelconso_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"chocolat","category":"food","date_from":"2026-10-01","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/fr_rappelconso_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-fr-rappelconso` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/fr-rappelconso/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "fr-rappelconso": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-fr-rappelconso"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Fr Rappelconso data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
