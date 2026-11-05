# @pipeworx/swappa-prices

Daily-updated used-device prices from Swappa: current active-listing averages
by variant, and realized (sold) average price per storage tier plus a sample
of individual sold listings with date/condition/price.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1866+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `search_device(query, limit?)` — find a device's Swappa slug by free-text
  name (needed by the other two tools, though they'll also auto-resolve a
  free-text `device` argument themselves).
- `device_prices(device)` — starting price, average (active-listing) price,
  and a variant breakdown table (carrier/storage → listing count, avg price,
  trade-in value).
- `sale_prices(device, condition?, carrier?, storage?)` — the realized SALE
  price average per storage tier (distinct from the list-price average in
  `device_prices`), plus a rolling sample of individual sold listings.

## Auth

Keyless. Public pages, no login required.

## Data sources

- `https://swappa.com/prices/<slug>` — per-device price page: starting/average
  price, variant table (`table.table-striped`), and a "Storage Prices" table
  (`#recent_prices`, columns Storage | Avg list price | Avg sale price) — the
  latter is the actual realized-sale-price data.
- `https://swappa.com/xui/product/<slug>/sales` — htmx partial backing the
  "Recent Sales" slide-out; a plain `HX-Request: true` header (no cookie/auth)
  is enough — without it the endpoint 302s. Accepts `condition`, `carrier`,
  `storage` query filters (values vary per device; discover them from the
  `<select>` options in the fragment, e.g. `unlocked`, `att`, `t-mobile`,
  `128gb`).
- `https://swappa.com/search/quick_search?q=<query>` — the site's own search
  autocomplete, used to resolve a free-text device name to its slug
  (`data-product-slug` attribute on `data-index-name="swappa-catalog-product"`
  links; `product_group` and `product_sub` entries are skipped — the former
  has no `/prices/` page, the latter carries an empty slug).

robots.txt (checked 2026-09-05) disallows only `/cgi-bin/` — no crawl-delay,
no anti-bot page block encountered on `/prices/*`, `/search/quick_search`, or
`/xui/product/*/sales`. Swappa's "Data Services" contact page 404'd during the
survey (contact-based bulk offering, unverified) — no bulk/API terms were
found linked from the `/prices` pages themselves during this build.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "swappa-prices": {
      "url": "https://gateway.pipeworx.io/swappa-prices/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/swappa-prices/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1866+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/search_device \
  -H 'Content-Type: application/json' \
  -d '{"query":"iPhone 15"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/search_device`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-swappa-prices` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/swappa-prices/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "swappa-prices": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-swappa-prices"]
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
ask_pipeworx({ question: "your question about Swappa Prices data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
