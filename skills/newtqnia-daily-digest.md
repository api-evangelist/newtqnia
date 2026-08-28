---
name: newtqnia-daily-digest
description: >-
  Fetch NewTqnia's bilingual technology-news digest over the public REST API and present
  it with the attribution the licence requires. Use when an agent needs current
  technology, AI, cybersecurity or science headlines in English or Arabic.
api: NewTqnia Daily Digest API
base_url: https://api.newtqnia.com
operations:
  - getTodaysNews
  - getLatestNews
generated: '2026-08-28'
method: generated
source: openapi/newtqnia-daily-digest-api.yaml + https://newtqnia.com/en/developers
---

# Fetch the NewTqnia daily digest

No credential is required. Both operations answered anonymously when probed.

## 1. Choose the collection

- **`getTodaysNews`** — `GET /v1/news/today`. Articles published today, where "today" is
  the **Asia/Dubai** calendar day, not UTC and not your local day. It returns HTTP 200
  with an **empty `articles` array** when nothing has been published yet today.
- **`getLatestNews`** — `GET /v1/news/latest`. The most recent articles regardless of
  day. **Prefer this when you need guaranteed content.**

A safe default: call `getTodaysNews`; if `articles` is empty, fall back to
`getLatestNews`.

## 2. Set the parameters

| Parameter | Values | Notes |
|---|---|---|
| `locale` | `en`, `ar` | Default `en`. Pass the user's language. |
| `limit` | 1–10 | Default 10. **10 is the hard ceiling** — there is no pagination. |
| `category` | a category slug | Optional. Slugs are listed at `https://newtqnia.com/en/categories`, e.g. `artificial-intelligence`, `cybersecurity`-adjacent `technology-policy`, `robotics`, `space`. |

```
GET https://api.newtqnia.com/v1/news/today?locale=en&limit=5
```

Optional identification headers — send them if you are a named application, they are not
required and do not unlock anything:

```
X-API-Key: ntq_...
X-NewTqnia-Application: my-dashboard
X-NewTqnia-Website: https://example.com
```

## 3. Read the response

The envelope tells you how to render itself. Read these before touching `articles`:

- `direction` is `ltr` or `rtl` — **use it**, do not infer direction from `locale`.
- `timezone` is always `Asia/Dubai`; `date` is that day.
- `api_version` is `v1`; assert it rather than assuming.
- `_links.self` echoes the exact query, `_links.documentation` and `_links.openapi` point
  at the docs and the contract.

Each `articles[]` entry has `id`, `title`, `summary`, `category{slug,name}`, `image`,
`url`, `published_at` and `read_time` (minutes).

## 4. Attribution is mandatory — do not skip this

The response carries its own licence terms in `attribution` and `usage`:

- Display **"Powered by NewTqnia"** as a **visible link** to `https://newtqnia.com`
  whenever you present this content.
- **Preserve `article.url` verbatim.** It is returned with
  `utm_source=newtqnia_api&utm_medium=api&utm_campaign=daily_digest_api`. Stripping the
  query string breaches section 9 of `https://newtqnia.com/en/terms`.
- Do **not** republish full article text. The API returns summaries only, by design.

## 5. Be a good client

- **Cache.** Responses carry `Cache-Control: max-age=300` and a weak `ETag`. Cache for
  five minutes and send `If-None-Match` on revalidation; a match returns `304` with no
  body.
- **Watch the budget.** Read `X-RateLimit-Limit` and `X-RateLimit-Remaining` from the
  response rather than trusting the documented number — the docs say 60 requests/minute
  per IP, but the live header advertised `20` when probed. **Trust the header.**
- **On `429`**, honor `Retry-After` and back off. Do not retry immediately.
- Every operation is a `GET`. Nothing here changes state, so retries are safe and there
  is nothing to undo.

## 6. When this API is not enough

The REST API only reaches **articles**, only the recent window, and only 10 at a time. If
you need a specific article by id, a story timeline, the bilingual glossary, or the
explainer library, use the MCP server instead — see `newtqnia-mcp-research`.
