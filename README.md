# @pipeworx/mfds-drugs

MFDS, South Korea's Ministry of Food and Drug Safety register of licensed medicines (식품의약품안전처 의약품통합정보): search by Korean or English brand name, English active ingredient or holder company, and get the full record for one product in **South Korea**.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2015+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Scope: South Korea only

For the US use `openfda`; EU-wide, `ema-medicines`; Australia, `tga-artg`; Singapore, `hsa-singapore`. Tools are prefixed `mfds_` so they never win a generic drug question.

## Tools

- `mfds_drug_search(query, by?, status?, limit?, page?)` — *Is Tylenol approved in Korea? Which Korean products contain acetaminophen? What does a given company hold?* Returns Korean and English brand name, holder, authorisation number and date, status (authorised / withdrawn / expired / cancelled / business closed as the register states it), main ingredient, ATC code, prescription or OTC class, domestic or import. 15 per page, 1-based `page`; the response carries `total`, `has_more` and `next_page`. Zero matches returns `found: false` with an `empty_reason`.
- `mfds_drug_get(id)` — the full record for one item code (품목기준코드): adds appearance, assessment type (generic or new drug), every active ingredient with amount and unit, excipients, barcodes, and links to the official efficacy, dosage, precautions and consumer-summary documents. Unknown code returns `found: false`.

Every response carries `source` and `data_as_of` (fetch time: the register is queried live per request and nothing is stored).

## Auth

Keyless. This pack does not use the registered-key open API; it reads the register's public search.

## Data sources

- MFDS (식품의약품안전처, Ministry of Food and Drug Safety), Drug Integrated Information System: the register of medicines licensed in South Korea.

### Things the next person would otherwise rediscover

- **There is no Korean active-ingredient search.** The ingredient filter is ignored for Korean text and returns the whole register (95,000+ rows). The pack therefore matches Korean text against the brand name (Korean product names usually contain the ingredient) and refuses a Korean `by: "ingredient"` with a message.
- **English ingredient names match as the register writes them** ("acetaminophen", not "paracetamol"). Brand-name matching is by substring.
- **The default includes every status** (authorised, withdrawn, expired, cancelled, business closed); use `status: "authorised"` for current licences only.
- **Status values stay as the register states them**; `status` is the English mapping and `status_register` the original Korean word.
- **Strength is only in the detail record**, not in search rows. The detail page is large (several hundred KB), so `mfds_drug_get` is a heavier call than search; if it cannot be read the record is still returned with a `note`.
- Page size is 15 and fixed upstream; `limit` can only trim a page.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mfds-drugs": {
      "url": "https://gateway.pipeworx.io/mfds-drugs/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mfds-drugs/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2015+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/mfds_drug_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"타이레놀"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/mfds_drug_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-mfds-drugs` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/mfds-drugs/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "mfds-drugs": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mfds-drugs"]
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
ask_pipeworx({ question: "your question about Mfds Drugs data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
