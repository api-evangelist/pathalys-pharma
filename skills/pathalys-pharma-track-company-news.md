---
name: pathalys-pharma-track-company-news
description: Monitor Pathalys Pharma's corporate news stream — financings, executive appointments, clinical milestones and scientific updates — from the company's own content API, without scraping HTML.
api: pathalys-pharma:pathalys-pharma-posts-api
operations:
  - listPosts
  - getPost
  - listCategories
---

# Track Pathalys Pharma company news

Pathalys Pharma publishes every press release and scientific update through the WordPress REST API
behind pathalys.com. Reading it is anonymous and needs no credentials. Use it instead of parsing the
news page: the HTML is Elementor-rendered and changes with the theme, the API does not.

Base URL: `https://pathalys.com/wp-json`

## Steps

1. **Establish the categories once.** `listCategories` — `GET /wp/v2/categories`. Pathalys uses
   exactly two: `News` (id 1) and `Scientific Updates` (id 11). Cache the ids; they are stable.

2. **Pull the stream.** `listPosts` — `GET /wp/v2/posts?per_page=100&orderby=date&order=desc&_fields=id,slug,date,modified,link,title,excerpt,categories`.
   Read `X-WP-Total` from the response headers to know the full count (14 at last measurement) and
   `X-WP-TotalPages` to know when to stop. Do not page past `X-WP-TotalPages` — that returns
   `rest_post_invalid_page_number` with HTTP 400.

3. **Poll incrementally after the first run.** Store the timestamp of your last successful pass and
   pass it as `modified_after`:
   `GET /wp/v2/posts?modified_after=2026-05-01T00:00:00&_fields=id,slug,modified,title,link`.
   `modified_after` catches corrections to existing releases as well as new ones; `after` catches
   only newly published items. Prefer `modified_after` for a monitor.

4. **Separate the two streams when it matters.** Regulatory and financing news is category 1;
   published upacicalcet science is category 11. Filter with `?categories=11` when you only want
   the scientific record.

5. **Fetch the body only for items you have not seen.** `getPost` — `GET /wp/v2/posts/{id}`. Take
   `content.rendered`; it is HTML, so strip tags before summarizing. Add `_embed` if you also want
   the featured image inlined rather than making a second call to the media API.

## Pacing and etiquette

- No rate-limit headers are returned by this API — you get no budget signal at all. Pathalys's
  robots.txt states `Crawl-delay: 10`; honour ten seconds between requests.
- Responses are edge-cached by WP Engine for 600 seconds (`cache-control: max-age=600`). Polling
  more often than every ten minutes returns you the same bytes.
- The content velocity is low: 14 posts between October 2020 and May 2026. A daily poll is generous;
  hourly is waste.

## Errors

Errors are the WordPress `WP_Error` envelope, **not** RFC 9457 problem+json:
`{"code":"rest_post_invalid_id","message":"Invalid post ID.","data":{"status":404}}`.
Branch on `code`, never on `message`. See `errors/pathalys-pharma-problem-types.yml`.

## Boundaries

- This surface is **read-only**. There is no write, no idempotency key and nothing to reverse.
- Do not attempt `context=edit` — it is rejected anonymously.
- `/wp/v2/settings` returns 401. Pathalys issues no credentials to third parties, so there is no
  path to satisfy it. Do not retry it.
