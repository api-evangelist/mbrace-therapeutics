---
name: mbrace-therapeutics-harvest-company-graph
description: Harvest the MBrace Therapeutics board, executive committee, founders, named investors and scientific advisory board as structured records from the five custom post types the company registered.
api: mbrace-therapeutics:mbrace-therapeutics-people-api
operations:
  - listBoardMembers
  - listExecutiveCommittee
  - listFounders
  - listInvestors
  - listScientificBoard
  - getMediaItem
  - listTypes
generated: '2026-08-25'
method: generated
source: openapi/mbrace-therapeutics-people-openapi.yml, data-model/mbrace-therapeutics-data-model.yml, conventions/mbrace-therapeutics-conventions.yml
---

# Harvest the MBrace Therapeutics company graph

MBrace Therapeutics registered five custom post types on its WordPress deployment, so its governance
and backer disclosures are queryable as JSON collections instead of scraped from an About page. This
is the one thing this API offers that a generic WordPress site does not.

Base URL: `https://mbracetrx.com/wp-json`. No credentials. All GETs. Five requests harvest the whole
graph.

## The five collections

| What | Route | operationId | Count at harvest |
|---|---|---|---|
| Board of directors | `/wp/v2/board-member` | `listBoardMembers` | 8 |
| Executive committee | `/wp/v2/executive-committee` | `listExecutiveCommittee` | 8 |
| Co-founders | `/wp/v2/founders` | `listFounders` | 2 |
| Institutional investors | `/wp/v2/investor` | `listInvestors` | 5 |
| Scientific advisory board | `/wp/v2/scientific-board` | `listScientificBoard` | 5 |

Counts were read from `X-WP-Total` on 2026-08-25.

## Harvest one collection

    GET /wp/v2/board-member?per_page=100&_fields=id,slug,title,content,link,featured_media,categories

Repeat for the other four routes. `per_page=100` covers every collection in a single page — none of
them exceeds eight items.

## Where the data actually is

There is no structured name, role or affiliation field. Read:

- `title.rendered` — the name, with post-nominals inline. Observed:
  `"Fabian Zohren, MD, PhD"`, `"Lillian L. Siu, MD, FRCPC, FASCO, FAACR"`,
  `"Renata Pasqualini, Ph.D."`, `"Sara Todd Sullivan"`.
- `content.rendered` — the biography as HTML. Role and affiliation are inside this prose.
- `featured_media` — an integer. Resolve with `getMediaItem` (`GET /wp/v2/media/{id}`) and read
  `source_url` for the headshot, or the investor's logo on the investor type.
- `link` — the public page for the record.

**The `acf` field is a trap.** Advanced Custom Fields is installed and every object carries an `acf`
key, but it returned an **empty array** on every collection sampled on 2026-08-25. Do not build a
parser that expects structured fields there. If a later run finds it populated, that is a real
change worth recording.

## Type asymmetries worth knowing

- `investor` is **not** attached to the `category` taxonomy and has **no `excerpt` field**. The other
  four types differ from each other here; do not assume one shape across all five.
- `board-member`, `executive-committee` and `founders` share the same four-term `category` taxonomy
  that classifies news posts, so a category value is not people-specific.
- `founders`, `board-member` and `scientific-board` carry a `parent` field; `executive-committee` and
  `investor` do not.

Confirm the current shape before a harvest with:

    GET /wp/v2/types

`listTypes`. Returns every registered post type with its `rest_base` and attached taxonomies. If
MBrace adds or removes a custom type, this is where it shows up first.

## Investors observed at harvest time

Blue Owl Capital, Avidity Partners, TPG, Venrock, Alta Partners. Treat the API as the source of
truth rather than this list — it is a snapshot of 2026-08-25.

## Rules

- Pace at one request per 10 seconds (`Crawl-delay: 10`), honour the 600-second cache, and send a
  realistic `User-Agent`.
- This is public information a visitor can read on the website. It is nonetheless data about named
  individuals: harvest the roster, not the people. Do not enrich it against other sources, and do
  not treat `/wp/v2/users` (3 site authors, anonymously readable) as part of this graph — it is
  staff account data and is deliberately excluded from the tool set in
  `mcp/mbrace-therapeutics-mcp.yml`.
- MBrace publishes no versioning or deprecation policy for this surface. These custom types exist at
  the discretion of a marketing CMS and can disappear with a theme change.
