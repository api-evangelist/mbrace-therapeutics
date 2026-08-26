---
name: mbrace-therapeutics-search-and-resolve
description: Find any published object on mbracetrx.com by keyword and resolve it to the full record, without guessing integer identifiers.
api: mbrace-therapeutics:mbrace-therapeutics-search-api
operations:
  - searchContent
  - getNewsPost
  - getPage
  - getBoardMember
  - getExecutiveCommitteeMember
  - getFounder
  - getInvestor
  - getScientificBoardMember
  - getMediaItem
generated: '2026-08-25'
method: generated
source: openapi/mbrace-therapeutics-search-openapi.yml, openapi/mbrace-therapeutics-people-openapi.yml, conventions/mbrace-therapeutics-conventions.yml
---

# Search and resolve on the MBrace Therapeutics content API

Object identifiers on this deployment are bare site-scoped integers with no type prefix. Never guess
one. Start at search, read `subtype`, then dereference against the collection that `subtype` names.

Base URL: `https://mbracetrx.com/wp-json`. No credentials. All GETs.

## Step 1 — search

    GET /wp/v2/search?search=<keyword>&per_page=20

`searchContent`. Returns lightweight references only:

    [{ "id": 4146, "title": "...", "url": "...", "type": "post", "subtype": "elementor_library" }]

76 objects were searchable at harvest time. `X-WP-Total` on the response tells you how many matched.

## Step 2 — pick the collection from `subtype`

`subtype` is the post type, and it maps directly to the route:

| subtype | route | operationId |
|---|---|---|
| `post` | `/wp/v2/posts/{id}` | `getNewsPost` |
| `page` | `/wp/v2/pages/{id}` | `getPage` |
| `attachment` | `/wp/v2/media/{id}` | `getMediaItem` |
| `board-member` | `/wp/v2/board-member/{id}` | `getBoardMember` |
| `executive-committee` | `/wp/v2/executive-committee/{id}` | `getExecutiveCommitteeMember` |
| `founders` | `/wp/v2/founders/{id}` | `getFounder` |
| `investor` | `/wp/v2/investor/{id}` | `getInvestor` |
| `scientific-board` | `/wp/v2/scientific-board/{id}` | `getScientificBoardMember` |

Search also returns internal `subtype` values such as `elementor_library`, which are page-builder
templates rather than content. Skip them.

## Step 3 — resolve, with a field list

    GET /wp/v2/board-member/4132?_fields=id,slug,title,content,link,featured_media

Always send `_fields`. Without it, `/wp/v2/pages` responses carry Elementor-rendered HTML in
`content.rendered` — a single page was roughly 200KB on the wire at harvest time against ~3KB for a
media item.

## Alternative — resolve by slug instead of id

Every collection accepts `?slug=`, and slugs are stable and human-readable where ids are not:

    GET /wp/v2/investor?slug=tpg

Prefer this when you already know the name of the thing you want.

## Rules

- Pace at one request per 10 seconds. `robots.txt` advertises `Crawl-delay: 10`, and it is the only
  pacing figure the site publishes. There are no rate-limit headers to read.
- Honour the cache. Collection responses carry `Cache-Control: max-age=600, must-revalidate`.
- Send a realistic `User-Agent`. A burst from a bare `curl` agent was answered with a Cloudflare 403
  HTML interstitial during profiling. If you receive HTML where you expected JSON, that is the edge
  challenge — back off, do not parse it as an error body.
- Branch on `code`, never on `message`. A 404 carries `rest_post_invalid_id` (bad or unpublished id)
  or `rest_no_route` (bad path); a 400 carries `rest_invalid_param` with the exact constraint in
  `data.details[<param>].message`. See `errors/mbrace-therapeutics-problem-types.yml`.
- Do not retry a 401. There is no self-service credential for this deployment.
