---
name: obi9-api
description: "Obi9's REST and WebSocket API for biotech companies: fully diluted share counts and enterprise value, SEC filings and earnings transcripts, clinical trials, FDA adverse events, and a feed of what just happened. WHEN: the user wants Obi9 data in a script, notebook, spreadsheet or backend job, mentions an Obi9 API key or an obi_ token, or names a URL under api.obi9.ai. Covers authenticating, the endpoint surface, and how to load the full reference in one call."
---

# Obi9 API

Programmatic access to the same numbers the Obi9 terminal shows. Plain REST
plus a WebSocket for alerts. Responses are CSV, text or JSON, so they drop into
a spreadsheet, a script or a backend job without an SDK.

## Authenticate

Base URL is `https://api.obi9.ai/api/v1`. Every request carries an API key in a
header, and the API is served over TLS only.

```bash
curl "https://api.obi9.ai/api/v1/fields?ticker=ACHV&field=SHARES_FD_TOTAL" \
  -H "X-API-Key: obi_YOUR_KEY"
```

Keys are created in the developer portal at
https://biotech.obi9.ai/developers. If the user does not have one, send them
there rather than guessing at another auth scheme: there is no password flow
and no OAuth on this surface.

## Read the reference before you construct anything non-obvious

One authenticated call returns the complete, current documentation as plain
text, every chapter of it:

```bash
curl "https://api.obi9.ai/api/v1/fields/docs" -H "X-API-Key: obi_YOUR_KEY"
```

Do this first when a request needs a field name, a parameter you have not used
before, or a response shape. It is cheaper and far more reliable than inferring
an endpoint from this file. The same text is the portal's "Full docs" button.

## The endpoint surface

Orientation only. The reference above is authoritative and current.

| Endpoint | What it answers |
| --- | --- |
| `GET /fields` | Valuation and share-count fields for one or many companies |
| `GET /fields/list` | Every field name you can ask for, with its meaning |
| `GET /fields/objects` | The object shapes the responses use |
| `GET /fields/search` | Full text across filings, transcripts, press releases, trial records, labels and regulatory letters |
| `GET /fields/feed` | What happened across the companies you follow, newest first, with the AI's analysis attached |
| `GET /fields/trials/search`, `GET /fields/trials/{id}` | Clinical trial records as registered, plus posted results |
| `GET /fields/catalysts` | The forecast catalyst calendar |
| `GET /fields/watchlists` | Watchlists you can see, and your own are editable |
| `GET /fields/versions` | Methodology version history, and what each version changed |
| `GET /fields/docs` | The full reference as plain text |
| `GET /fields/skills` | Task-level guidance, see below |
| `wss://api.obi9.ai/api/ws?api_key=obi_YOUR_KEY` | Real-time alert notifications |

## Deeper guidance, fetched live

`GET /fields/skills` lists task-level skills by name and description, and
`GET /fields/skills/{name}` returns one as markdown. They cover things this
file deliberately does not: which call to make for a given question, how to
read the answer, and the traps in each data family.

Prefer them over reasoning from first principles. They are served from the same
source the Obi9 assistants use, so they are current in a way an installed file
cannot be.

## Three things that cost people time

**Identify a company with the parameter that matches what you hold.** Pass
`ticker_bbg=` for Bloomberg form, `obi_id=` for an Obi9 identifier, `ticker=`
for a plain symbol. Guessing the wrong one is the most common first error.

**Never report a share count or a valuation without its basis.** Fully diluted
and treasury-stock-method counts answer different questions and can differ
substantially on the same company at the same instant. Anything
price-dependent also needs the price it was computed at. State both rather than
handing over a bare number.

**A pinned `data_version` freezes methodology, not data.** Numbers under any
pin still move as new filings arrive. If you need to explain why a figure
changed, `GET /fields/versions` tells you whether methodology moved.

## Anything the AI produced is the AI's reading

Analysis text, summaries and significance ratings in the feed and elsewhere are
produced by a model. Present them as the AI's judgement of a document, never as
an established fact about the company and never as advice. Quote the underlying
document when the distinction matters. Every response links the source it came
from, so the document is always one hop away.

## Availability

Not every capability is enabled on every plan. If a credential cannot reach
something, the response says so; ask Obi9 to enable it rather than working
around it.

Full documentation: https://biotech.obi9.ai/developers
