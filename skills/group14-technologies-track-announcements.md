---
name: group14-technologies-track-announcements
description: Track new Group14 Technologies press releases, news coverage, blog posts and whitepapers from the company's own content API, without scraping HTML.
api: group14-technologies:group14-technologies-resources-api
operations:
  - listResources
  - getResource
  - listResourceCategories
generated: '2026-08-22'
method: generated
source: openapi/group14-technologies-resources-openapi.yml
---

# Track Group14 Technologies announcements

Group14 publishes no press API and no developer program. It does expose the WordPress REST
content API behind `group14.technology`, anonymously and read-only, and that is the cleanest
machine-readable route to its announcements. 169 resources at capture on 2026-08-22.

Base URL: `https://group14.technology/wp-json`. No credentials. No key. No signup.

## 1. Resolve the category you care about

`listResourceCategories` — `GET /wp/v2/resource-category?per_page=10&_fields=id,name,slug,count`

At capture the vocabulary was: Press Releases (`31`, 42), Blog (`33`, 36), News (`34`, 89),
Whitepapers (`29`, 5), Company News (`32`, 0).

**Resolve the id every run.** Term ids are WordPress database keys; the `slug` is the stable
thing. Never hardcode `31` — look it up by `slug: press-releases`.

## 2. Pull what is new

`listResources` — `GET /wp/v2/resource?resource-category=<id>&after=<ISO8601>&orderby=date&order=desc&per_page=100&_fields=id,slug,link,date,modified,title,excerpt,resource-category`

- `after` takes an ISO 8601 datetime and is the incremental hook. Store the highest `date` you
  have seen and pass it back next run.
- `modified_after` catches edits to records you already have — worth a second pass if you care
  about corrections to press releases.
- `per_page` maxes at 100; going over returns `400 rest_invalid_param`.

## 3. Page correctly

Read `X-WP-Total` and `X-WP-TotalPages` from the response headers and stop at `X-WP-TotalPages`.
Walking past the last page returns `400 rest_post_invalid_page_number`, not an empty array. There
is also an RFC 8288 `Link: …; rel="next"` header if you would rather follow cursors.

## 4. Fetch the full body only when you need it

`getResource` — `GET /wp/v2/resource/{id}` returns `content.rendered` as HTML. The list call with
`_fields` is far cheaper; only drop to the item call for records you actually intend to read.

## Rules

- **Pace against the cache.** Responses carry `cache-control: max-age=600, must-revalidate`. There
  are no rate-limit headers and no published limits, so 600 seconds is the only budget signal
  Group14 gives you. Polling faster gains nothing and risks a Cloudflare edge block.
- **Branch on content-type before parsing.** Cloudflare and WP Engine sit in front of the origin.
  An edge rejection arrives as HTML, not as the WordPress JSON envelope.
- **Errors are not RFC 9457.** The shape is `{"code":"…","message":"…","data":{"status":…}}`.
  Branch on `code`. See `errors/group14-technologies-problem-types.yml`.
- **Read-only, always.** Write methods exist on these routes but return `401 rest_forbidden` with
  no public credential path. Never attempt one.
- **Resolve by `slug`, cache by `slug`.** Numeric ids are database keys and carry no guarantee.
