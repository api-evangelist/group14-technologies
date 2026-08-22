---
name: group14-technologies-map-manufacturing-footprint
description: Read Group14 Technologies' factory and office footprint — BAM-1, BAM-2, BAM-3, the silane factory and the Seoul offices — as structured records rather than from a map graphic.
api: group14-technologies:group14-technologies-locations-api
operations:
  - listLocations
  - getLocation
  - listPages
  - getPage
generated: '2026-08-22'
method: generated
source: openapi/group14-technologies-locations-openapi.yml
---

# Map the Group14 manufacturing footprint

Group14's `location` custom post type is the single most company-specific thing on its public
API surface: the battery active materials (BAM) factory network as data. 6 published locations at
capture on 2026-08-22.

Base URL: `https://group14.technology/wp-json`. Anonymous, read-only.

## 1. List the facilities

`listLocations` — `GET /wp/v2/location?per_page=100&_fields=id,slug,link,title,featured_media,acf`

At capture this returned Seoul Offices (`seouloffices`), BAM-1, BAM-2, BAM-3 and the Group14
silane factory. `X-WP-Total: 6`.

## 2. Read the detail

`getLocation` — `GET /wp/v2/location/{id}` returns `content.rendered`.

Facility specifics — capacity, status, geography — live in the `acf` object (Advanced Custom
Fields). **The server declares `acf` as a free-form object with no advertised properties.** Do not
assume a field name. Inspect a live record, and treat any field you find as unversioned: it can
change whenever the site is redesigned, and nothing announces that.

## 3. Cross-reference the manufacturing pages

The narrative lives in the page tree, not the location records:

`listPages` — `GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent`

`/manufacturing/` is the parent; `/manufacturing/bam-1/`, `/manufacturing/bam-2/`,
`/manufacturing/bam-3/` and `/manufacturing/silane-factory/` are its children, linked by the
`parent` field. Follow `parent` to rebuild the hierarchy rather than parsing URLs.

## 4. Join the announcements

Factory milestones are announced through the resource library, not the location records. Use
`listResources` with `search=BAM` (see `group14-technologies-track-announcements`) to pick up
openings, expansions and capacity news, then join on the facility name in the title.

## Rules

- **Two sources of truth, deliberately.** `location` records are the structured footprint; pages
  are the narrative. Neither is complete alone, and they are not guaranteed to agree.
- **Never invent an `acf` field.** If it is not in a live response, it does not exist.
- **Language matters here.** Every route accepts `wpml_language` (`en`, `ko`, `ja`, `de`,
  `zh-hans`). The Korean BAM-3 site content may differ between `en` and `ko`; request explicitly
  rather than accepting the default.
- **Pace against `cache-control: max-age=600`.** No rate-limit headers are published.
