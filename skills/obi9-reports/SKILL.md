---
name: obi9-reports
description: "Installs and authenticates the Obi9 reports connector, then uses it: the clinical-trial and FDA adverse-event data behind the reader's Obi9 emails, plus control of the reports themselves, which watchlist leads them and what is on their shortlist. WHEN: the user asks you to add, install, connect, fix or re-authenticate the Obi9 connector, or receives Obi9 reports and asks about a trial, an adverse-event report, or changing what they receive. Covers the install command per client, the browser sign-in, what to do when it fails, and how to get task-level guidance once attached."
---

# Obi9 reports connector: install, authenticate, use

Puts the data behind a reader's daily Obi9 reports inside the AI assistant they
already use, and lets them change what arrives. Your first job is usually to
get the connector attached. Do that before answering the question they actually
asked.

## 1. Install

The address is:

```
https://biotech.obi9.ai/mcp-reports
```

It speaks MCP over Streamable HTTP with OAuth, so any spec-compliant client can
attach. Use the one that matches where you are running.

**Claude Code**

```bash
claude mcp add --transport http obi9 https://biotech.obi9.ai/mcp-reports
```

Then run `/mcp` in Claude Code and pick Obi9 to start the sign-in. Add
`--scope user` to the add command if the user wants it available in every
project rather than just this one.

**Claude desktop or claude.ai**

Settings, then Connectors, then Add custom connector, and paste the address.

**Other clients**

Anything that supports a remote MCP server over Streamable HTTP will work.
Point it at the address above as an HTTP (not stdio) server; in editors that
keep an `mcp.json`, that is an entry carrying the `url`. Follow the client's own
documentation for the exact key names, which change more often than this file
can track.

This is the narrower of the two Obi9 connectors. A reader with a full Obi9
terminal account wants the `obi9-terminal` skill instead, which also reaches
companies, filings, catalysts, valuation and market data.

## 2. Authenticate

Nothing to create beforehand. Do NOT go looking for a client ID, a client
secret or an API key: the client registers itself with Obi9 automatically and
authentication happens in the browser.

What the user will see, in order:

1. The client opens a browser to the Obi9 sign-in page. If it cannot open one
   it prints the URL instead; tell the user to open it.
2. They sign in. Either their normal Obi9 password, or **Email me a sign-in
   code**, which sends a **6-digit** code to their address. The code option
   exists for report readers who have no terminal password, which is most of
   them.
3. They approve the connection for their own account.
4. The browser hands the client its token and the connection is live.

**There is no code to copy back into the terminal.** If you are used to GitHub's
device flow, where the CLI shows an eight-character code to type into a web
page, this is not that. Obi9 uses the authorization code flow with PKCE and a
loopback redirect, so the credential returns to the client automatically. The
only code anywhere in this flow is the 6-digit email one, and it is typed into
the browser, not into your terminal.

The token is per user, and it acts on that person's own reports, so never share
one.

## 3. When it does not work

- **Tools listed but every call returns unauthorized.** The connector is
  registered but not signed in. Re-run the sign-in (`/mcp` in Claude Code).
- **Sign-in page opens but the client never receives the token.** The redirect
  comes back to a loopback address on the user's machine. A VPN or a proxy that
  intercepts localhost will break it. Retry off the VPN.
- **The client insists on a client ID or secret.** It has not found our
  discovery documents. Confirm the address is exactly the one above, with no
  trailing slash or extra path, and that the machine can reach
  `biotech.obi9.ai`.
- **The client needs a fixed callback port.** Claude Code accepts
  `--callback-port <port>` on the add command.
- **The email code never arrives.** It goes to the address Obi9 sends the
  reports to. If they are reading reports, that is the address to use.

## 4. What the toolset reaches

Three groups, and you do not need to name tools or memorise options.

- **Clinical trials.** US and EU registry records, searched and read, plus how a
  record changed over time and the documents an EU trial has posted.
- **Adverse events.** FDA FAERS activity for a drug, company or watchlist, and
  the individual case reports behind the counts, which is where the actual
  content is.
- **Your reports.** Everything that shapes the emails: which reports arrive,
  which watchlist leads them, what is on the shortlist.

## 5. Get the real guidance from the connector

The connector serves task-level skills of its own, and they are the useful
layer: which tool answers a given question, how to read what comes back, and
what a number does and does not mean. Read the relevant one before improvising
a sequence of calls.

Ask the connector to list its skills, or call its `read_skill` tool. They are
served live, so they stay current in a way this installed file cannot.

## 6. Before you answer

**Report tools write to a real account.** Changing reports, the leading
watchlist or the shortlist changes what a person receives by email, and the
effect is immediate. Read the current state before changing it, and say what
you changed.

**An adverse-event count is not an incidence rate.** FAERS reports are
voluntarily submitted. A count reflects what was reported, not what happened in
a population, and it moves with attention as much as with events. Say so when a
count is the answer, and open the case reports when the detail matters.

**A registry record is what the sponsor registered.** It is current and
complete as a mirror of the registry, and it is not independent verification of
what a trial did.

**Anything the AI produced is the AI's reading.** Analysis text, summaries and
significance ratings come from a model. Present them as the AI's judgement of a
document, never as an established fact about the company and never as advice.
The underlying document is always linked and is what settles a disagreement.

## Availability

Not every capability is enabled on every plan. If a credential cannot reach
something, the reply says so; ask Obi9 to enable it rather than working around
it.

Full documentation: https://biotech.obi9.ai/mcp-docs
