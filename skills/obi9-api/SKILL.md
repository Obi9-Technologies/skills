---
name: obi9-api
description: "Drives the Obi9 REST and WebSocket API for biotech companies: share counts and enterprise value, SEC filings and transcripts, clinical trials, medical conference abstracts, FDA adverse events, search and a feed. WHEN: the user wants Obi9 data in a script, notebook, spreadsheet or backend job, mentions an Obi9 API key, OBI9_API_KEY or an obi_ token, names a URL under api.obi9.ai, or is getting 401 or 403 from Obi9. Covers setup and the call that verifies it, reading just the documentation chapter a task needs, the parameter conventions, and managing API keys and the IP allowlist."
---

# Obi9 API

Programmatic access to the same numbers the Obi9 terminal shows. Plain REST
plus a WebSocket for alerts. Responses are CSV, text or JSON, so they drop into
a spreadsheet, a script or a backend job without an SDK.

This is the key-authenticated surface. If the user wants Obi9 inside their AI
assistant instead, that is a connector, not this: see the `obi9-terminal` skill.

## 1. Setup

This file is the whole install: save it as `obi9-api/SKILL.md` in your agent's
skills folder (`~/.cursor/skills` in Cursor, `~/.claude/skills` in Claude
Code). There is nothing else to install and no sign-in step: the API
authenticates with a key alone.

- **The key comes from the environment.** Read it from `OBI9_API_KEY` and send
  it in the `X-API-Key` header. Never print it, and never ask the user to paste
  it into the chat, where it would stay in the transcript. If the variable is
  empty, ask the user to set it (`export OBI9_API_KEY=obi_...` on macOS or
  Linux, `setx OBI9_API_KEY obi_...` on Windows) and restart the editor or
  terminal, which reads the environment only when it starts.
- **No key yet?** Keys are created in the developer portal (section 4).
- **Keys work only from registered IP addresses** (section 5). A new account
  has to register one before any key answers.

## 2. Verify

One call proves the key, the allowlist and the plan together:

```bash
curl -s -w "\nHTTP %{http_code}\n" -H "X-API-Key: $OBI9_API_KEY" \
  "https://api.obi9.ai/api/v1/fields?ticker_bbg=ACHV%20US%20Equity&field=SHARES_FD_TOTAL"
```

In PowerShell, call `curl.exe` and write the key as `$env:OBI9_API_KEY`.

| Status | Meaning | Fix |
| --- | --- | --- |
| `200` and a number | Setup works | Nothing |
| `401` | The key is missing, mistyped, expired or revoked. Most often `OBI9_API_KEY` is not set in the environment the agent runs in. | Set it there and restart the editor or terminal |
| `403` | The key is fine but this caller is not allowed. The message says why, usually that this IP address is not registered. | Register the address (section 5) |

Tell the user the status you got. Report setup as done only on a `200`.

## 3. Read only what the task needs

The full reference is long, and reading all of it before a first call spends
much of your context. Load the part the task needs:

- `GET /api/v1/fields/docs?chapter=<id>` returns one chapter of the reference
  as plain text. Chapters: `getting-started` (authentication, rate limits,
  errors), `fields-data` (fields and the field catalog), `watchlists`,
  `search`, `feed`, `trials`, `conferences`, `faers`, `objects` (response
  shapes), `websocket` and `support`. An unknown id answers 404 with the
  current list.
- `GET /api/v1/fields/skills` lists task-level guidance by name and
  description, and `GET /api/v1/fields/skills/{name}` returns one as markdown:
  which call answers a given question, how to read the answer, and the traps
  in each data family.
- `GET /api/v1/fields/list?ticker_bbg=ACHV%20US%20Equity` names every field
  available for that company, with its meaning and a live value. It needs a
  company.
- `GET /api/v1/fields/objects` gives the object shapes the responses use.
- `GET /api/v1/fields/docs` with no chapter returns every chapter at once, for
  when the task really needs all of it.

The same documentation for people is at https://biotech.obi9.ai/developers.
Each chapter id works as an anchor there (`#faers`, `#support`), so you can
send the user straight to the part that answers them.

## 4. Managing keys

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
- Keep the key in `OBI9_API_KEY`, never in client-side code, a committed file
  or a chat. It carries the owner's own entitlements.

Monitoring, under the portal's Monitoring section, shows recent requests with
status codes per key. That is the fastest way to tell a bad key from a bad
request: if the call never appears there, it never arrived.

## 5. The IP allowlist

The allowlist is in the portal next to API Keys:

- **Keys work only from registered addresses.** Until the organization
  registers at least one, every key answers **403**. This is the most common
  "my key does not work" on a new account.
- It covers **every key anyone in the organization holds**, now and later, so
  an address one colleague registers works for everyone's keys.
- Requests from any other address are rejected with **403**.
- Exact IPv4 or IPv6 addresses only. No ranges, no CIDR, no hostnames.

So when a call 403s, check the allowlist before suspecting the key. And when a
user moves to a new network, a VPN, a CI runner or a new cloud region, their
egress address changed and needs adding. A laptop on a home connection often
has an address that changes on its own, which makes a static allowlist
frustrating; say so rather than letting them re-add it repeatedly without
understanding why.

## 6. Parameter conventions

The reference is authoritative, but these are the ones that trip people on the
first call.

**Identify a company in Bloomberg form.** Pass `ticker_bbg=ACHV%20US%20Equity`
(that is `ACHV US Equity`, URL-encoded). `ticker=` also takes Bloomberg form or
an Obi9 id, and `obi_id=` takes an Obi9 id such as `OBI0000646`. Pass exactly
one of them. A bare symbol like `ACHV` can match more than one listing, so do
not rely on it.

**Ask for the fields you want.** `field=` takes one or several, comma
separated. `GET /fields/list?ticker_bbg=...` is how you find out what exists.

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

## 7. The endpoint surface

Orientation only. The chapters in section 3 have parameters and response
shapes.

| Endpoint | What it answers |
| --- | --- |
| `GET /fields` | Valuation and share-count fields for one or many companies |
| `GET /fields/list?ticker_bbg=...` | Every field available for one company, with its meaning and a live value |
| `GET /fields/objects` | The object shapes responses use |
| `GET /fields/search` | Full text across filings, transcripts, press releases, trial records, labels and regulatory letters |
| `GET /fields/feed` | What happened across the companies you follow, newest first, with the AI's analysis attached |
| `GET /fields/trials/search`, `GET /fields/trials/{id}` | Trial records as registered, plus posted results |
| `GET /fields/catalysts` | The forecast catalyst calendar |
| `GET /fields/conferences`, `GET /fields/conferences/abstracts` | Medical meetings with every key date, and their abstracts, searchable by words or by meaning |
| `GET /fields/faers`, `/fields/faers/reactions`, `/fields/faers/cases` | FDA adverse-event reports: counts and cases, the cases reporting a reaction, one case in full |
| `GET /fields/watchlists` | Watchlists you can see; your own are editable |
| `GET /fields/versions` | Methodology version history |
| `GET /fields/docs` | The reference as plain text: one chapter with `chapter=`, or all of it |
| `POST /fields/support`, `GET /fields/support/{ticket}` | Report a failing call or a wrong answer to the Obi9 team (`text`, optionally `endpoint` and `status`); the answer is a ticket whose status, outcome and replies you can check, comment on (`POST .../{ticket}/comments`) or ask to close (`POST .../{ticket}/close`) |
| `GET /fields/skills` | Task-level guidance (section 3) |
| `wss://api.obi9.ai/api/ws?api_key=$OBI9_API_KEY` | Real-time alert notifications |

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
