---
name: pathalys-pharma-research-upacicalcet
description: Answer questions about upacicalcet and Pathalys Pharma's chronic kidney disease programme from primary company sources, using the site's search API rather than a web crawl.
api: pathalys-pharma:pathalys-pharma-search-api
operations:
  - searchContent
  - getPost
  - getPage
  - listPosts
---

# Research upacicalcet from Pathalys's primary sources

When a question concerns upacicalcet, secondary hyperparathyroidism (SHPT), or Pathalys Pharma's
clinical programme, the company's own statements are the primary source. This skill retrieves them
directly instead of relying on press aggregation.

Base URL: `https://pathalys.com/wp-json`

## Steps

1. **Search across the whole estate.** `searchContent` —
   `GET /wp/v2/search?search=upacicalcet&subtype=any&per_page=100`. Returns a flat projection over
   posts and pages: `{id, title, url, type, subtype}`. A search for `upacicalcet` matched 17 objects
   at last measurement. `X-WP-Total` gives the true count.

2. **Resolve each hit against the right collection.** `subtype` tells you which: `post` → `getPost`
   (`GET /wp/v2/posts/{id}`), `page` → `getPage` (`GET /wp/v2/pages/{id}`). The ids share one
   namespace, so using the wrong collection returns `rest_post_invalid_id` with HTTP 404.

3. **Prefer the dated record for claims about the programme.** `listPosts` with
   `?categories=11` returns the Scientific Updates stream; `?categories=1` returns News, which is
   where financings, the Launch Therapeutics collaboration, the Phase 3 announcement and executive
   appointments live. Cite the post `date` and `link`, not the search projection.

4. **Read the disease and mechanism framing from the Home page.** `getPage` on id 174 carries
   Pathalys's own description of the SHPT problem, the calcimimetic mechanism, the pre-filled-syringe
   end-of-dialysis administration, and the Catalys Pacific / DaVita Venture Group founding.

## Accuracy rules that matter here

- **Always carry the regulatory qualifier.** Pathalys states on its own home page that upacicalcet is
  an investigational product candidate in the United States, subject to review and approval by the
  FDA. Any summary that omits this misrepresents the company's position.
- The site is not a clinical data source. It publishes no trial data, no protocol, and no CDISC or
  FHIR interface. For registrational detail, go to the trial registries, not this API.
- `search` is a substring match, not a ranked relevance engine. Two searches with different casing
  or partial stems will return different sets; search the full term.

## Errors and pacing

- Error envelope is WordPress `WP_Error` (`code` / `message` / `data.status`), not RFC 9457.
- No rate-limit headers. Honour `Crawl-delay: 10` from robots.txt; responses are edge-cached 600s.

## Boundaries

Read-only surface. Nothing here creates, modifies or spends anything, so there is no reversal path
to consider and no confirmation to seek before acting.
