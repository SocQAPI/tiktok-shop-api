<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# TikTok Shop API

[![SocQ TikTok Shop API](https://img.shields.io/badge/API-SocQ%20TikTok%20Shop%20API-F64C31)](https://socq.ai/apis/tiktok-shop?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/tiktok-shop/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/tiktok-shop-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/tiktok-shop-api/actions/workflows/check.yml)

9 endpoints for public TikTok Shop data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse TikTok Shop APIs](https://socq.ai/apis/tiktok-shop?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api) · [Documentation](https://docs.socq.ai/api-manual/tiktok-shop/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [TikTok Shop Search API](#tiktok-shop-search-api) | `tiktok-shop/search` | Search public TikTok Shop products by keyword and market region, returning normalized product, seller, price, media, rating, review, and sold-count fields. | 0.7 credits ($0.0035) per result | 2026-07-20 |
| [TikTok Shop Products API](#tiktok-shop-products-api) | `tiktok-shop/products` | Collect public products from one TikTok Shop store URL, returning identity, prices, images, seller context, ratings, reviews, and sold counts as normalized records. | 0.7 credits ($0.0035) per result | 2026-07-20 |
| [TikTok Shop Product API](#tiktok-shop-product-api) | `tiktok-shop/product` | Retrieve one public TikTok Shop product from a PDP URL, returning normalized product identity, seller, price, visible commerce signals, media, categories, and SKU data. | 0.7 credits ($0.0035) per result | 2026-07-20 |
| [TikTok Shop Product Reviews API](#tiktok-shop-product-reviews-api) | `tiktok-shop/product-reviews` | Collect public reviews for one TikTok Shop product from its PDP URL, returning normalized review text, ratings, reviewer details, images, SKU context, and purchase-status fields. | 0.3 credits ($0.0015) per result | 2026-07-20 |
| [TikTok Shop User Showcase API](#tiktok-shop-user-showcase-api) | `tiktok-shop/user-showcase` | Collect products displayed in one public TikTok user's Shop showcase, returning normalized product, seller, price, media, rating, review, and sold-count fields. | 0.7 credits ($0.0035) per result | 2026-07-20 |
| [TikTok Shop Hot Products API](#tiktok-shop-hot-products-api) | `tiktok-shop/hot-products` | Collect the hot-selling products of a TikTok Shop market with price, units sold, rating and shop. | 0.5 credits ($0.0025) per result | 2026-11-17 |
| [TikTok Shop Categories API](#tiktok-shop-categories-api) | `tiktok-shop/categories` | Get the full product category tree of a TikTok Shop market with IDs, levels and parents. | 0.5 credits ($0.0025) per call | 2026-11-17 |
| [TikTok Shop Category Products API](#tiktok-shop-category-products-api) | `tiktok-shop/category-products` | Browse the products of a TikTok Shop category with price, units sold, rating, reviews and shop. | 0.5 credits ($0.0025) per result | 2026-11-17 |
| [TikTok Shop Search Suggestions API](#tiktok-shop-search-suggestions-api) | `tiktok-shop/search-suggestions` | Get the search completions TikTok Shop suggests for the start of a shopping query. | 0.5 credits ($0.0025) per call | 2026-11-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/tiktok-shop-api.git
cd tiktok-shop-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs tiktok-shop/search

# Python 3.8+ (standard library only)
python3 examples/python/socq.py tiktok-shop/search payloads/search.json

# cURL + jq
bash examples/curl/socq.sh tiktok-shop/search '{"query":"running shoes","results_limit":100,"region":"US"}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### TikTok Shop Search API

Search public TikTok Shop products by keyword and market region, returning normalized product, seller, price, media, rating, review, and sold-count fields.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/search`
- **publicId:** `tiktok-shop/search`
- **Price:** 0.7 credits ($0.0035) per result
- **Launched:** 2026-07-20
- **Example body:** [`payloads/search.json`](payloads/search.json)

Search public products by keyword and market region.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `query` | string | Yes | Search query. |
| `region` | string | No | - |
| `results_limit` | integer | No | (value 1 to 2000) |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "running shoes",
  "results_limit": 100,
  "region": "US"
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/search](https://docs.socq.ai/api-manual/tiktok-shop/search) · Landing page: [socq.ai/apis/tiktok-shop/search](https://socq.ai/apis/tiktok-shop/search?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

### TikTok Shop Products API

Collect public products from one TikTok Shop store URL, returning identity, prices, images, seller context, ratings, reviews, and sold counts as normalized records.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/products`
- **publicId:** `tiktok-shop/products`
- **Price:** 0.7 credits ($0.0035) per result
- **Launched:** 2026-07-20
- **Example body:** [`payloads/products.json`](payloads/products.json)

Collect products from one store URL, sorted by best sellers or newest releases.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `region` | string | No | - |
| `results_limit` | integer | No | (value 1 to 2000) |
| `sort_by` | string | No | - |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/products" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.tiktok.com/shop/store/cmf-by-nothing/7495669706651568260",
  "results_limit": 20
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/products](https://docs.socq.ai/api-manual/tiktok-shop/products) · Landing page: [socq.ai/apis/tiktok-shop/products](https://socq.ai/apis/tiktok-shop/products?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

### TikTok Shop Product API

Retrieve one public TikTok Shop product from a PDP URL, returning normalized product identity, seller, price, visible commerce signals, media, categories, and SKU data.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/product`
- **publicId:** `tiktok-shop/product`
- **Price:** 0.7 credits ($0.0035) per result
- **Launched:** 2026-07-20
- **Example body:** [`payloads/product.json`](payloads/product.json)

Collect product details, seller data, pricing, categories, media, and SKU information.

This endpoint currently supports US only. Omit region or set it to US; other regions are rejected before task creation.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `region` | string | No | Currently available for the US region only. Other regions are unavailable for this endpoint. (one of `US`; default `"US"`) |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/product" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.tiktok.com/shop/pdp/1729548220307968132"
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/product](https://docs.socq.ai/api-manual/tiktok-shop/product) · Landing page: [socq.ai/apis/tiktok-shop/product](https://socq.ai/apis/tiktok-shop/product?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

### TikTok Shop Product Reviews API

Collect public reviews for one TikTok Shop product from its PDP URL, returning normalized review text, ratings, reviewer details, images, SKU context, and purchase-status fields.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/product-reviews`
- **publicId:** `tiktok-shop/product-reviews`
- **Price:** 0.3 credits ($0.0015) per result
- **Launched:** 2026-07-20
- **Example body:** [`payloads/product-reviews.json`](payloads/product-reviews.json)

Collect public review text, ratings, reviewer information, images, and purchase status.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `region` | string | No | - |
| `results_limit` | integer | No | (value 1 to 2000) |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/product-reviews" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.tiktok.com/shop/pdp/1729548220307968132",
  "results_limit": 20
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/product-reviews](https://docs.socq.ai/api-manual/tiktok-shop/product-reviews) · Landing page: [socq.ai/apis/tiktok-shop/product-reviews](https://socq.ai/apis/tiktok-shop/product-reviews?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

### TikTok Shop User Showcase API

Collect products displayed in one public TikTok user's Shop showcase, returning normalized product, seller, price, media, rating, review, and sold-count fields.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/user-showcase`
- **publicId:** `tiktok-shop/user-showcase`
- **Price:** 0.7 credits ($0.0035) per result
- **Launched:** 2026-07-20
- **Example body:** [`payloads/user-showcase.json`](payloads/user-showcase.json)

Collect products displayed in one public TikTok user's Shop showcase.

This endpoint currently supports US only. Omit region or set it to US; other regions are rejected before task creation. results_limit must be 20 to 2000.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `region` | string | No | Currently available for the US region only. Other regions are unavailable for this endpoint. (one of `US`; default `"US"`) |
| `results_limit` | integer | No | (value 20 to 2000) |
| `username` | string | Yes | A public account username. |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/user-showcase" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "username": "simplymandys",
  "results_limit": 20
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/user-showcase](https://docs.socq.ai/api-manual/tiktok-shop/user-showcase) · Landing page: [socq.ai/apis/tiktok-shop/user-showcase](https://socq.ai/apis/tiktok-shop/user-showcase?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

### TikTok Shop Hot Products API

Collect the hot-selling products of a TikTok Shop market with price, units sold, rating and shop.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/hot-products`
- **publicId:** `tiktok-shop/hot-products`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/hot-products.json`](payloads/hot-products.json)

See what is selling on TikTok Shop in a given market. Products come back in the order TikTok Shop ranks them, each with its title, price and currency, units sold, rating, review count, shop and product images. Run it weekly per market to spot breakout products, benchmark prices, or pick winners to source and promote.

Provide a market in region: ID, JP, MX, MY, PH, SG, TH or VN. results_limit defaults to 50 and accepts integers from 1 to 200. Each product costs 0.5 credits.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `region` | string | Yes | Country or region code of the TikTok Shop market. (one of `ID`, `JP`, `MX`, `MY`, `PH`, `SG`, `TH`, `VN`) |
| `results_limit` | integer | No | (value 1 to 200; default `50`) |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/hot-products" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "region": "MY",
  "results_limit": 50
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/hot-products](https://docs.socq.ai/api-manual/tiktok-shop/hot-products) · Landing page: [socq.ai/apis/tiktok-shop/hot-products](https://socq.ai/apis/tiktok-shop/hot-products?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

### TikTok Shop Categories API

Get the full product category tree of a TikTok Shop market with IDs, levels and parents.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/categories`
- **publicId:** `tiktok-shop/categories`
- **Price:** 0.5 credits ($0.0025) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/categories.json`](payloads/categories.json)

Get the complete category tree of a TikTok Shop market in one call. Each category is a row with its ID, name, English slug, level, parent and the names above it, so the hierarchy can be rebuilt as a tree or used as a flat lookup. Use the IDs with TikTok Shop Category Products to browse a category's listings, or map the tree to your own catalog for market research.

region is optional and defaults to US; US, SG, MY, PH, TH, VN, ID, JP and MX are available. Each call costs 0.5 credits and returns every category of that market.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `region` | string | No | Country or region code of the TikTok Shop market. (one of `ID`, `JP`, `MX`, `MY`, `PH`, `SG`, `TH`, `US`, `VN`; default `"US"`) |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/categories" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "region": "US"
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/categories](https://docs.socq.ai/api-manual/tiktok-shop/categories) · Landing page: [socq.ai/apis/tiktok-shop/categories](https://socq.ai/apis/tiktok-shop/categories?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

### TikTok Shop Category Products API

Browse the products of a TikTok Shop category with price, units sold, rating, reviews and shop.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/category-products`
- **publicId:** `tiktok-shop/category-products`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/category-products.json`](payloads/category-products.json)

Browse a TikTok Shop category the way shoppers do. Pass a category ID and get its products with title, price, units sold, rating, reviews, shop and images. Pair it with TikTok Shop Categories to sweep a whole vertical, compare price bands between subcategories, or watch how a category's best sellers change over time.

Provide a category ID from TikTok Shop Categories in category_id. region defaults to US; US, SG, MY, PH, TH, VN, ID, JP and MX are available. results_limit defaults to 30 and accepts integers from 1 to 500. Each product costs 0.5 credits.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `category_id` | string | Yes | Numeric category ID, as returned by TikTok Shop Categories. (pattern `^[0-9]{1,20}$`) |
| `region` | string | No | Country or region code of the TikTok Shop market. (one of `ID`, `JP`, `MX`, `MY`, `PH`, `SG`, `TH`, `US`, `VN`; default `"US"`) |
| `results_limit` | integer | No | (value 1 to 500; default `30`) |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/category-products" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "category_id": "601450",
  "region": "US",
  "results_limit": 30
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/category-products](https://docs.socq.ai/api-manual/tiktok-shop/category-products) · Landing page: [socq.ai/apis/tiktok-shop/category-products](https://socq.ai/apis/tiktok-shop/category-products?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

### TikTok Shop Search Suggestions API

Get the search completions TikTok Shop suggests for the start of a shopping query.

- **Request:** `POST https://api.socq.ai/v1/tiktok-shop/search-suggestions`
- **publicId:** `tiktok-shop/search-suggestions`
- **Price:** 0.5 credits ($0.0025) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/search-suggestions.json`](payloads/search-suggestions.json)

See how shoppers finish a search on TikTok Shop. Type the start of a query and get up to 20 completions in the order TikTok Shop suggests them. Suggestions come from what people actually search, so they surface product variants, compatible models and buying angles worth targeting in listings, ads and content.

Provide the start of a shopping search in query. region defaults to US; US, SG, MY, PH, TH, VN, ID, JP and MX are available. Each call costs 0.5 credits and returns up to 20 completions ranked from 1.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `query` | string | Yes | Search query. |
| `region` | string | No | Country or region code of the TikTok Shop market. (one of `ID`, `JP`, `MX`, `MY`, `PH`, `SG`, `TH`, `US`, `VN`; default `"US"`) |

```bash
curl -X POST "https://api.socq.ai/v1/tiktok-shop/search-suggestions" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "phone case",
  "region": "US"
}'
```

Full docs: [https://docs.socq.ai/api-manual/tiktok-shop/search-suggestions](https://docs.socq.ai/api-manual/tiktok-shop/search-suggestions) · Landing page: [socq.ai/apis/tiktok-shop/search-suggestions](https://socq.ai/apis/tiktok-shop/search-suggestions?utm_source=github&utm_medium=readme&utm_campaign=socq-tiktok-shop-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible TikTok Shop data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
