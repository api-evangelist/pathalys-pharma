---
name: pathalys-pharma-mirror-corporate-content
description: Mirror the Pathalys Pharma corporate estate — pages, media and brand assets — into a local index for search, citation or asset use, using the site's own content API.
api: pathalys-pharma:pathalys-pharma-pages-api
operations:
  - listPages
  - getPage
  - listMedia
  - getMediaItem
  - getSiteIndex
---

# Mirror the Pathalys Pharma corporate estate

Build a faithful local copy of what Pathalys Pharma says about itself — the seven corporate pages and
the 104-item media library — without scraping. Everything below is anonymous.

Base URL: `https://pathalys.com/wp-json`

## Steps

1. **Read the site index first.** `getSiteIndex` — `GET /wp-json/`. It returns the site name
   (`Pathalys Pharma`), the home URL, the registered namespaces, the full route table with argument
   definitions, and `site_logo` (media id 205). Treat this document as the contract: if a route you
   want is not in it, it does not exist on this install.

2. **List the pages.** `listPages` — `GET /wp/v2/pages?per_page=100&_fields=id,slug,title,link,date,modified`.
   Seven are published: Home (174), Team (255), News (264), Scientific Updates (587), Contact us
   (161), Privacy Policy (295), Terms of Use (294). All are top-level; `parent` is 0 throughout, so
   there is no hierarchy to walk.

3. **Fetch each page body.** `getPage` — `GET /wp/v2/pages/{id}`. `content.rendered` is
   Elementor-generated HTML and carries inline `<style>` fragments; strip tags and collapse
   whitespace before indexing, or you will index CSS as prose.

4. **Enumerate the media library.** `listMedia` —
   `GET /wp/v2/media?per_page=100&_fields=id,slug,media_type,mime_type,source_url,alt_text`.
   Page through using `X-WP-TotalPages`. `source_url` is the direct file under
   `/wp-content/uploads/`; fetch that, not `link`, which is the attachment landing page.

5. **Resolve brand assets by id, not by guess.** `getMediaItem` — `GET /wp/v2/media/205` returns the
   Pathalys wordmark (`image/svg+xml`,
   `https://pathalys.com/wp-content/uploads/2021/09/pathalys_logo.svg`). The favicon is exposed
   directly as `site_icon_url` in the site index.

## Efficiency

- Always send `_fields`. The unfiltered page payload is large because it embeds rendered Elementor
  markup; a listing pass rarely needs `content`.
- Use `_embed` on a page or post request to inline the featured media instead of issuing a second
  media call per item.
- Respect the 600-second edge cache and the `Crawl-delay: 10` in robots.txt. A full mirror of this
  estate is roughly 120 requests; at ten seconds apart that is twenty minutes, and it only needs
  doing once.

## Licensing caution

The API serves the content without asserting a licence. Pathalys's Terms of Use
(https://pathalys.com/terms-of-use/) govern reuse of the text and imagery, and upacicalcet is an
investigational product candidate in the United States — any republication of the science should
carry that qualifier, which the company itself attaches on the home page.

## Boundaries

- Read-only. No writes, no reversals, no idempotency concerns.
- `/wp/v2/users` is anonymously readable but is deliberately excluded from this profile and from
  these skills under the enrichment PII guardrail. Do not harvest it.
