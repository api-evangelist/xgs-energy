---
name: xgs-energy-read-news-archive
description: Read and page through the XGS Energy news and insights archive (108 published posts) over the anonymous WordPress REST content API, resolving categories and featured images.
api: xgs-energy:xgs-energy-posts-api
operations:
  - listPosts
  - getPost
  - listCategories
  - getMediaItem
generated: '2026-09-04'
method: generated
source: openapi/xgs-energy-posts-api-openapi.yml, openapi/xgs-energy-categories-api-openapi.yml, openapi/xgs-energy-media-api-openapi.yml
---

# Read the XGS Energy news archive

XGS Energy publishes no developer program. The only callable surface is the WordPress REST content
API behind `https://www.xgsenergy.com/wp-json`. It is anonymous, read-only, undocumented by the
company, and can change without notice — do not build anything durable on it.

## Steps

1. **List posts.** Call `listPosts` — `GET /wp/v2/posts?per_page=100&page=1&orderby=date&order=desc`.
   No credential is required.
2. **Page to the end.** Read `X-WP-Total` (108 at capture) and `X-WP-TotalPages` from the response
   headers, or follow the `rel="next"` entry in the RFC 8288 `Link` header. `per_page` is capped at
   100; asking for more returns `400 rest_invalid_param`.
3. **Trim the payload.** Add `_fields=id,date,slug,link,title,excerpt,categories,featured_media` so
   you are not carrying rendered HTML you do not need. Add `_embed` instead when you want the
   category terms and featured image inlined in one round trip.
4. **Filter by date or topic.** Use `after` / `before` (ISO 8601) for a window, and `categories=<id>`
   with ids from `listCategories` (4 terms at capture) for a topic.
5. **Fetch one post.** Call `getPost` — `GET /wp/v2/posts/{id}`. An unknown id returns
   `404 rest_post_invalid_id`.
6. **Resolve the image.** If `featured_media` is non-zero, call `getMediaItem` —
   `GET /wp/v2/media/{featured_media}` — and read `source_url` or a specific entry under
   `media_details.sizes`.

## Rules

- **Read-only.** Every write method in the underlying route index is authentication-gated. There is
  no idempotency mechanism here and nothing to reverse, because there is nothing to write.
- **No rate limits are published.** No `X-RateLimit-*`, `RateLimit-*` or `Retry-After` header is
  returned. The origin sits behind Cloudflare and WP Engine, so back off politely on your own —
  responses are cached `max-age=600`, so re-reading faster than that gains you nothing.
- **Errors are not problem+json.** Expect `{"code": "...", "message": "...", "data": {"status": N}}`.
  Branch on `code`, not on the message text.
- **Never call `/wp/v2/users`.** It answers anonymously with author personal data. It is out of scope
  for this skill on purpose.
