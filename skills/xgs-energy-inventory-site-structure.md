---
name: xgs-energy-inventory-site-structure
description: Inventory the machine-readable surface XGS Energy actually exposes — route index, content types, taxonomies, media library and oEmbed — before assuming an API exists.
api: xgs-energy:xgs-energy-discovery-api
operations:
  - getRouteIndex
  - listContentTypes
  - listTaxonomies
  - listCategories
  - listTags
  - listMedia
  - getOEmbed
generated: '2026-09-04'
method: generated
source: openapi/xgs-energy-discovery-api-openapi.yml, openapi/xgs-energy-categories-api-openapi.yml, openapi/xgs-energy-tags-api-openapi.yml, openapi/xgs-energy-media-api-openapi.yml, openapi/xgs-energy-oembed-api-openapi.yml
---

# Inventory the XGS Energy machine-readable surface

Start here before assuming XGS Energy has an API. It does not sell one. What follows is how to see
exactly what is there, from the host's own self-description.

## Steps

1. **Get the route index.** Call `getRouteIndex` — `GET /wp-json/`. This one document names the site,
   the registered namespaces (21 at capture), the authentication schemes, and every route (266 at
   capture) with per-argument types, defaults and enums. Every OpenAPI in this repository was derived
   from it.
2. **Separate public from gated.** Most of those 266 routes refuse anonymous callers. Probe before
   you document: `/wp/v2/settings` and `/wp/v2/menus` return `401`, `/wp-abilities/v1/abilities`
   returns `401 rest_forbidden`, and `/hfe/v1/mcp-abilities` returns `403 uae_rest_not_allowed`.
   Only 17 operations are anonymously usable, and those are the ones documented here.
3. **List the content model.** Call `listContentTypes` (`GET /wp/v2/types`) and `listTaxonomies`
   (`GET /wp/v2/taxonomies`) to see what the site models, then `listCategories` (4 terms) and
   `listTags` (1 term) for the actual vocabulary.
4. **Inventory assets.** Call `listMedia` — `GET /wp/v2/media?per_page=100` — and page with
   `X-WP-TotalPages` (182 attachments at capture).
5. **Get embeddable metadata.** Call `getOEmbed` —
   `GET /oembed/1.0/embed?url=https%3A%2F%2Fwww.xgsenergy.com%2F` — for oEmbed 1.0 rich metadata for
   any post or page URL.

## Rules

- **Do not report the presence of an MCP server.** The `wp-abilities` and `hfe/v1/mcp-abilities`
  routes are WordPress administrator plumbing, both auth-gated. XGS Energy publishes no agent
  endpoint, and `/.well-known/agent-card.json`, `/.well-known/agent.json` and `/.well-known/mcp.json`
  all return 404.
- **Do not enumerate `/wp/v2/users`.** It answers anonymously with author personal data and is
  excluded from every skill and tool in this repository on purpose.
- Read-only throughout. No writes, no idempotency surface, nothing to reverse.
