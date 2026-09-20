---
name: obi9-api
description: "Drives the Obi9 REST and WebSocket API for biotech companies: share counts and enterprise value, SEC filings and transcripts, clinical trials, FDA adverse events, search and a feed. WHEN: the user wants Obi9 data in a script, notebook, spreadsheet or backend job, mentions an Obi9 API key or an obi_ token, names a URL under api.obi9.ai, or is getting 401 or 403 from Obi9. Covers loading the full reference in one call, the parameter conventions, and managing API keys and the IP allowlist."
---

# Obi9 API

Programmatic access to the same numbers the Obi9 terminal shows. Plain REST
plus a WebSocket for alerts. Responses are CSV, text or JSON, so they drop into
a spreadsheet, a script or a backend job without an SDK.

This is the key-authenticated surface. If the user wants Obi9 inside their AI
assistant instead, that is a connector, not this: see the `obi9-terminal` skill.

## 1. Read the reference before you construct anything

Do this FIRST, not as a fallback. One authenticated call returns the complete,
current documentation as plain text, every chapter of it:

```bash
curl "https://api.obi9.ai/api/v1/fields/docs" -H "X-API-Key: obi_YOUR_KEY"
```

It is the same text the portal's "Full docs" button copies, and it is
authoritative in a way this file cannot be. Load it whenever a request needs a
field name, a parameter you have not used, or a response shape. Guessing an
endpoint from the table below and then debugging a 400 is slower than reading.

Two more discovery calls worth knowing:

- `GET /api/v1/fields/list` names every field you can ask for, with its meaning.
- `GET /api/v1/fields/objects` gives the object shapes the responses use.

The human-readable version is at https://biotech.obi9.ai/developers, organised
as chapters: Quickstart, Fields and Data, Watchlists, Search, Feed, Trials,
Objects and WebSocket. Each chapter's anchor works as a deep link, so you can
send the user straight to the part that answers them.

## 2. Authenticate

Base URL `https://api.obi9.ai/api/v1`. Every request carries the key in a
header, and the API is TLS only.

```bash
curl "https://api.obi9.ai/api/v1/fields?ticker=ACHV&field=SHARES_FD_TOTAL" \
  -H "X-API-Key: obi_YOUR_KEY"
```

There is no password flow and no OAuth on this surface. Keys look like
`obi_...` and are created in the developer portal, covered below.

## 3. Managing keys

Keys live in the portal at https://biotech.obi9.ai/developers, under API Keys.
The portal is where they are created, named and revoked; there is no endpoint
that mints a key, so send the user there rather than hunting for one.

What to tell a user who is setting up:

- The key is shown once, at creation. If they did not copy it, revoke it and
  create another rather than trying to recover it.
- Name keys after where they run, because the portal reports usage per key and
  an unnamed key is impossible to retire safely later.
- Revoking is immediate. Rotate by creating the replacement, moving traffic,
  then revoking the old one.
- Never put a key in client-side code or a committed file. It carries the
  owner's own entitlements.

Monitoring, under the portal's Monitoring section, shows recent requests with
status codes per key. That is the fastest way to tell a bad key from a bad
request: if the call never appears there, it never arrived.

## 4. The IP allowlist, which is the usual reason for a 403

Read this before debugging anything. The allowlist is in the portal next to API
Keys, and it behaves in a way that surprises people:

- It is **account-level**, not per key. It applies to every key on the account,
  the ones that exist now and any created later.
- **An account with no registered address cannot use its keys at all.** A brand
  new key on a fresh account will fail until an address is added. This is the
  single most common "my key does not work".
- Requests from anywhere else are rejected with **403**.
- Exact IPv4 or IPv6 addresses only. No ranges, no CIDR, no hostnames.

So when a call 403s, check the allowlist before suspecting the key. And when a
user moves to a new network, a VPN, a CI runner or a new cloud region, their
egress address changed and needs adding. A laptop on a home connection often
has an address that changes on its own, which makes a static allowlist
frustrating; say so rather than letting them re-add it repeatedly without
understanding why.

`401` means the key is missing, malformed or revoked. `403` means the key is
fine but the caller is not allowed, which in practice means the allowlist.

## 5. Parameter conventions

The reference is authoritative, but these are the ones that trip people on the
first call.

**Identify a company with the parameter matching what you hold.** Pass
`ticker_bbg=` for Bloomberg form (for example `ACHV US Equity`), `obi_id=` for
an Obi9 identifier, or `ticker=` for a plain symbol. Using the wrong one is the
most common first error.

**Ask for the fields you want.** `field=` takes one or several, comma
separated. `GET /fields/list` is how you find out what exists.

**Choose the response shape.** `format=` selects between CSV, plain text and
JSON. Scalars and tables default to CSV, which is what makes this pleasant from
a spreadsheet.

**Paginate with the cursor you were given.** Endpoints that page return a
`next_cursor`; pass it back rather than computing offsets.

**Pin a methodology version if reproducibility matters.** `data_version=`
freezes the methodology a number was computed with.
`GET /fields/versions` lists the history and what each version changed. Note
that a pin freezes METHODOLOGY, not data: numbers still move under any pin as
new filings arrive.

## 6. The endpoint surface

Orientation only. Load the reference for parameters and response shapes.

| Endpoint | What it answers |
| --- | --- |
| `GET /fields` | Valuation and share-count fields for one or many companies |
| `GET /fields/list` | Every field name, with its meaning |
| `GET /fields/objects` | The object shapes responses use |
| `GET /fields/search` | Full text across filings, transcripts, press releases, trial records, labels and regulatory letters |
| `GET /fields/feed` | What happened across the companies you follow, newest first, with the AI's analysis attached |
| `GET /fields/trials/search`, `GET /fields/trials/{id}` | Trial records as registered, plus posted results |
| `GET /fields/catalysts` | The forecast catalyst calendar |
| `GET /fields/watchlists` | Watchlists you can see; your own are editable |
| `GET /fields/versions` | Methodology version history |
| `GET /fields/docs` | The full reference as plain text |
| `GET /fields/skills` | Task-level guidance, see below |
| `wss://api.obi9.ai/api/ws?api_key=obi_YOUR_KEY` | Real-time alert notifications |

## 7. Deeper guidance, fetched live

`GET /fields/skills` lists task-level skills by name and description, and
`GET /fields/skills/{name}` returns one as markdown. They cover what this file
deliberately does not: which call to make for a given question, how to read the
answer, and the traps in each data family. Served from the same source the Obi9
assistants use, so they stay current.

## 8. Two things to get right in an answer

**Never report a share count or a valuation without its basis.** Fully diluted
and treasury-stock-method counts answer different questions and can differ
substantially on the same company at the same instant. Anything
price-dependent also needs the price it was computed at.

**Anything the AI produced is the AI's reading.** Analysis text, summaries and
significance ratings in the feed come from a model. Present them as the AI's
judgement of a document, never as an established fact about the company and
never as advice. Every response links its source, so the document is one hop
away.

## Availability

Not every capability is enabled on every plan. If a credential cannot reach
something, the response says so; ask Obi9 to enable it rather than working
around it.

Full documentation: https://biotech.obi9.ai/developers
