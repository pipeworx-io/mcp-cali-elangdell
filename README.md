# @pipeworx/cali-elangdell

CALI eLangdell Press's free, Creative Commons-licensed US law-school casebooks
and course materials: 62 titles (torts, contracts, constitutional law,
evidence, civil procedure, tax, legal research, property, criminal law …) with
authors, topics, PDF/EPUB/Word download links on cali.org, and the exact
licence read from each book's own front matter.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1851+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `elangdell_titles(topic?, license?, limit?)` — the catalogue, filterable by
  a topic word or a licence id/prefix; every response carries
  `titles_by_license` so the licence mix is visible.
- `elangdell_search(query, limit?)` — titles by words in the title, an author
  name or the description.
- `elangdell_book(slug)` — one title in full: downloads, suggested attribution,
  and the per-title licence as the `license` envelope.

## Auth

Keyless. The pack serves from a curated allowlist (`src/titles.ts`); it makes
no upstream call. Downloads link to CALI's own files.

## Licence — the allowlist is the compliance story

Licence varies **per title**, and the bookstore pages do not say which: the
only statement is inside each book. `src/titles.ts` was generated on
2026-09-14 by reading the front matter of every title on
<https://www.cali.org/the-elangdell-bookstore> (64 listed): the EPUB where
there is one, else the PDF, else the Word file. Each row records the file the
licence was read from and the book's own "Suggested attribution format" line.

| Licence read from the book | Titles |
|---|---|
| CC BY-NC-SA 4.0 | 33 |
| CC BY-NC-SA 3.0 | 23 |
| CC BY-SA 4.0 | 3 |
| CC BY-SA 3.0 | 3 |

Two of the 64 are deliberately **not listed** (`NOT_LISTED` in `titles.ts`,
with the reason): *Civil Procedure: Pleading* (2014), whose EPUB, PDF and
Word files contain no Creative Commons statement at all, and *CALI Lessons: A
Guidebook…*, which has no downloadable file. Where a file names more than one
licence — cover-art and photo credits under other CC terms, or an EPUB
metadata field left over from a prior edition — the row takes the licence in
the book's own "This work … is licensed and published by CALI eLangdell Press
under a Creative Commons …" sentence and says so in `license_resolution`.

Two things the brief assumed that the books contradict: *Sources of American
Law* is **BY-NC-SA 4.0** in its current (8th) edition, not BY-SA; the six
BY-SA titles are *Torts: Cases and Contexts* vols 1–2 (4.0), *Distance
Learning in Legal Education* (4.0), and three Evidence titles by Colin Miller
(3.0: Rule 404, Rule 410, *Selected Materials on the Law of Evidence*).

Consequences (fleet #1974, decision #1973):

- `zeroRated: true` on the gateway entry — 0 credits on every tier — because
  the NonCommercial titles forbid charging. The BY-SA rows would not need it;
  `requires_zero_rating` on `elangdell_book` says which.
- Every response leads with `attribution`, `license` and `license_note` via
  `attachLicense` in `@pipeworx/shared`; list responses carry the strictest
  terms any row has (BY-NC-SA), `elangdell_book` carries the title's own.
- Nothing is ingested: no book text is retained, only the catalogue rows.

## Data sources

- <https://www.cali.org/the-elangdell-bookstore?page=0..2> — the catalogue
  (28 cards per page, three pages, some titles repeated across pages).
- <https://www.cali.org/books/{slug}> — a title's page: description, topics,
  `field--name-field-ebook-downloads` links. **The book pages answer a browser
  User-Agent with an Anubis proof-of-work challenge** (a JavaScript bot wall);
  they answer an honest non-browser UA such as
  `PipeworxBot/1.0 (+https://pipeworx.io)` directly. The pack does not fetch
  them at all; the survey did, one page at a time with a 1.5 s pause.
- <https://www.cali.org/sites/default/files/…> — the EPUB/PDF/DOCX files, no
  wall. The licence statement is in the OPF `dc:rights` field and the
  "Notices" page; when the two disagree the Notices page is the one that names
  the edition.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "cali-elangdell": {
      "url": "https://gateway.pipeworx.io/cali-elangdell/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/cali-elangdell/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1851+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/elangdell_titles \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/elangdell_titles`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "cali-elangdell": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-cali-elangdell"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-cali-elangdell
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Cali Elangdell data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
