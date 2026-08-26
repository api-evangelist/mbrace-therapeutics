---
name: mbrace-therapeutics-track-news-archive
description: Read and monitor the MBrace Therapeutics news archive — press releases, conference presentations and publications — as structured JSON rather than scraped HTML.
api: mbrace-therapeutics:mbrace-therapeutics-news-api
operations:
  - listNews
  - getNewsPost
  - listCategories
  - getMediaItem
generated: '2026-08-25'
method: generated
source: openapi/mbrace-therapeutics-news-openapi.yml, openapi/mbrace-therapeutics-taxonomy-openapi.yml, conventions/mbrace-therapeutics-conventions.yml
---

# Track the MBrace Therapeutics news archive

The company news at `https://mbracetrx.com/news/` is also readable as JSON. Use the API — the HTML
is Elementor-rendered and the JSON is not.

Base URL: `https://mbracetrx.com/wp-json`. No credentials. All GETs.

## Read the archive

    GET /wp/v2/posts?per_page=100&_fields=id,slug,title,excerpt,date,modified,link,categories

`listNews`. 8 posts at harvest time (2026-08-25), the most recent dated 2025-07-04. Read `X-WP-Total`
for the count and `X-WP-TotalPages` for the page count; the `Link` header carries `rel="next"`.

The archive at harvest time covered: emergence from stealth with the $85M Series B led by TPG,
preclinical data at SABCS 2023 and AACR 2024, the appointment of Steve Alley as CSO, initiation of
patient dosing in the MBRC-101 Phase 1, the ASCO 2024 trial-in-progress poster, and the
first-generation EphA5-targeted ADC publication.

## Poll for new items

    GET /wp/v2/posts?after=<last-seen-ISO8601>&orderby=date&order=asc&_fields=id,slug,title,date,link

`after` filters on publication date; use `modified_after` instead if you also care about edits to
existing posts. An empty array means nothing new.

Poll at most daily. This archive gained a handful of items across three years, responses are cached
for 600 seconds, and `robots.txt` asks for `Crawl-delay: 10`.

## Get the full body of one item

    GET /wp/v2/posts/{id}

`getNewsPost`. `content.rendered` is HTML. `excerpt.rendered` is a short summary and is usually
enough — request it via `_fields` before pulling full bodies.

## Categories

    GET /wp/v2/categories?per_page=100

`listCategories`. 4 terms at harvest time. On this deployment the category taxonomy is shared with
the `board-member`, `executive-committee` and `founders` types, so a category is not necessarily a
news category. The `post_tag` taxonomy is registered but empty — do not filter on `tags`.

## Featured images

Each post carries `featured_media` as an integer. Resolve it with `getMediaItem`
(`GET /wp/v2/media/{id}`) and read `source_url`, or add `_embed` to the list request to inline it
under `_embedded['wp:featuredmedia']` and save a round trip.

## Rules

- No credentials, and none available. Do not retry a 401.
- Branch on the `code` field of an error body, not the message.
- Send a realistic `User-Agent`; an HTML body where JSON was expected is the Cloudflare edge
  challenge, not an application error.
- This is a marketing CMS, not a regulated disclosure feed. It is not a substitute for
  ClinicalTrials.gov, an SEC filing or a press-wire feed, and MBrace commits to nothing about its
  availability or completeness.
