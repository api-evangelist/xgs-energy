---
name: xgs-energy-search-site-content
description: Search across all XGS Energy website content (127 searchable objects) and resolve each hit to its full post or page record.
api: xgs-energy:xgs-energy-search-api
operations:
  - search
  - getPost
  - getPage
  - listContentTypes
generated: '2026-09-04'
method: generated
source: openapi/xgs-energy-search-api-openapi.yml, openapi/xgs-energy-posts-api-openapi.yml, openapi/xgs-energy-pages-api-openapi.yml, openapi/xgs-energy-discovery-api-openapi.yml
---

# Search XGS Energy site content

Use this when you need to answer a question about XGS Energy from its own words — its technology,
projects, partnerships or leadership — rather than from third-party coverage.

## Steps

1. **Know what is searchable.** Call `listContentTypes` — `GET /wp/v2/types` — to see which content
   types exist. At capture the searchable set is posts and pages, 127 objects in total.
2. **Search.** Call `search` — `GET /wp/v2/search?search=<terms>&per_page=100`. Optionally narrow
   with `subtype=post` or `subtype=page`.
3. **Read the projection.** Each hit is deliberately thin: `id`, `title`, `url`, `type`, `subtype`.
   It is a pointer, not the content.
4. **Resolve the hit.** Call `getPost` for `subtype: post` and `getPage` for `subtype: page`, using
   the `id` from the hit. Read `content.rendered` for the body and `date` for recency.
5. **Cite the public URL.** Use the `link` field of the resolved record, never the `/wp-json/` URL,
   when quoting XGS Energy back to a person.

## Rules

- Anonymous. No key, no signup, no OAuth.
- `per_page` is capped at 100; over that returns `400 rest_invalid_param`.
- Page with `page` plus the `X-WP-Total` / `X-WP-TotalPages` headers.
- `content.rendered` is HTML. Strip it before handing it to a model as prose.
- This is a website search, not a product API. It carries no availability commitment of any kind.
