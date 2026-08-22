---
name: group14-technologies-monitor-hiring
description: Read Group14 Technologies open roles and the department and location vocabularies behind them, as a hiring-signal feed for the silicon battery materials sector.
api: group14-technologies:group14-technologies-careers-api
operations:
  - listJobOpenings
  - getJobOpening
  - listJobDepartments
  - listJobLocations
generated: '2026-08-22'
method: generated
source: openapi/group14-technologies-careers-openapi.yml
---

# Monitor Group14 Technologies hiring

Group14's careers page is backed by a `job-opening` custom post type with two taxonomies. This is
usable as a demand signal: which factory is staffing, and for what discipline. 4 open roles, 19
departments and 19 job locations at capture on 2026-08-22.

Base URL: `https://group14.technology/wp-json`. Anonymous, read-only.

## 1. Pull the vocabularies first

- `listJobDepartments` — `GET /wp/v2/job-department?per_page=100&_fields=id,name,slug,count`
  (People & Culture, Quality Control, Maintenance, Engineering, Supply Chain, …)
- `listJobLocations` — `GET /wp/v2/job-location?per_page=100&_fields=id,name,slug,count`
  (Sangju split by function, Seoul, Woodinville, …)

The `count` on each term is the signal on its own: it tells you where hiring is concentrated
without reading a single posting.

## 2. Pull the roles

`listJobOpenings` — `GET /wp/v2/job-opening?per_page=100&orderby=date&order=desc&_fields=id,slug,link,date,modified,title,job-department,job-location`

Filter with `job-department=<id>` or `job-location=<id>` using ids resolved in step 1 — never
hardcoded.

## 3. Detect closures, not just openings

A filled role is *removed* from the collection; nothing announces it. Diff the set of `slug`
values between runs. A slug that disappeared is a role that closed — which is often the more
interesting signal than a new posting.

## 4. Note the second surface

Group14 also runs a Greenhouse board at `https://job-boards.greenhouse.io/group14`. It is a
separate system and the two do not necessarily agree. If completeness matters, read both and
reconcile on title plus location; do not assume the WordPress copy is authoritative.

## Rules

- **Diff on `slug`, not `id`.** Ids are database keys.
- **`acf` is free-form.** Compensation, requisition ids or employment type may or may not be
  present. Inspect; never assume.
- **Respect the people involved.** These are public job postings. Do not enrich them against
  individuals, and note that `/wp/v2/users` is deliberately gated (`401`) on this host — author
  records are not public and must not be pursued.
- **Pace against `cache-control: max-age=600`.** No rate-limit headers are published.
