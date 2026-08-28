---
name: newtqnia-mcp-research
description: >-
  Connect to NewTqnia's hosted MCP server and research its newsroom — recent articles, a
  specific article, a developing-story timeline, the bilingual technology glossary, and
  the explainer library. Use when the REST digest is too narrow or too shallow.
api: NewTqnia MCP
endpoint: https://newtqnia.com/mcp
operations:
  - server_status
  - get_recent_news
  - get_news_by_id
  - get_timeline_by_id
  - search_terminology
  - search_explainers
generated: '2026-08-28'
method: generated
source: mcp/newtqnia-mcp-tools.json (live tools/list) + https://newtqnia.com/en/connect
---

# Research NewTqnia over MCP

The MCP server reaches **four content types** the REST API does not expose, searches
full text, and windows back as far as 3,650 days. It is the right surface for research;
the REST digest is the right surface for a headline feed.

## 1. Get a credential first

Discovery is anonymous, but **invoking a tool requires a credential**.

- **API key (simplest).** Sign in at `https://newtqnia.com`, create a key from your
  account page, and copy it immediately — the full `ntq_...` value is shown **once**. You
  may hold up to 5 active keys and can label, replace or revoke any of them.
- Send it as `X-API-Key: ntq_...`, or as `Authorization: Bearer ntq_...` if your client
  only offers a bearer-token field.
- **OAuth 2.1 + PKCE** is available for clients that cannot send a custom header (some
  ChatGPT connector setups). Discovery:
  `https://newtqnia.com/.well-known/oauth-protected-resource`. Scopes: `mcp:read`,
  `mcp:write`.

Client config:

```json
{ "mcpServers": { "newtqnia": {
    "url": "https://newtqnia.com/mcp",
    "headers": { "X-API-Key": "ntq_..." } } } }
```

## 2. Connect

Streamable HTTP. Call `initialize`, then read **`Mcp-Session-Id`** from the response
headers and send it on **every** subsequent request. Without it the server returns
JSON-RPC `-32600`: *"A valid session id is REQUIRED for non-initialize requests."*

Negotiated protocol version at probe time: `2025-11-25`.

## 3. Pass `locale` on every call

Every content tool takes `locale` (`en` | `ar`, default `en`). Pass the user's language
explicitly. A reader token always receives **one** locale; only administrator tokens get
both.

## 4. Pick the tool

| Tool | Use it for | Key inputs |
|---|---|---|
| `get_recent_news` | A window of published articles. **Far wider than REST**: `days` 1–3650 (default 90), `limit` 1–500 (default 100). | `days`, `limit`, `locale` |
| `get_news_by_id` | One article by its exact numeric id. There is no REST equivalent. | `news_id` (required), `locale` |
| `get_timeline_by_id` | A developing story as a dated, sourced event sequence. | `timeline_id` (required), `locale` |
| `search_terminology` | The bilingual glossary. Matches English term, Arabic term, abbreviation or slug, case-insensitively. | `query` (required, ≤255), `limit` 1–50, `locale` |
| `search_explainers` | Evergreen explainers. Matches title, slug, summary or content, in both languages. | `query` (required, ≤255), `limit` 1–50, `locale` |
| `server_status` | Health check. Returns `{status, application, timestamp}`. | none |

`search_terminology` and `search_explainers` are the **only full-text search** NewTqnia
exposes to a machine.

## 5. Know what you will get back

Every tool is annotated `readOnlyHint: true`, `idempotentHint: true`,
`destructiveHint: false`. **Nothing you can call changes state**, so retries are safe and
there is nothing to reverse.

A reader token receives **title, summary and a canonical URL** — **never the article
body**. If a user needs the full text, give them the returned `url`; do not try to
reconstruct the article from summaries. Article ids and timeline ids are separate
integer namespaces, so an id from one is meaningless in the other.

`get_news_by_id` also returns a `citations[]` array of `{title, url}` — the newsroom's
source trail. Use it when a user asks "where did this come from?"

## 6. Cite it

The same attribution rule applies as on the REST API: show **"Powered by NewTqnia"** with
a visible link, and preserve the canonical URLs the tools return.
`https://newtqnia.com/en/terms`
