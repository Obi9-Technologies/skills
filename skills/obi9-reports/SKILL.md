---
name: obi9-reports
description: "Connects your assistant to the Obi9 reports connector over MCP: the clinical-trial and FDA adverse-event data behind your Obi9 emails, plus control of the reports themselves, which watchlist leads them and what is on your shortlist. WHEN: the user receives Obi9 reports and asks about a trial, an adverse-event report, or changing what they receive, or asks to add or fix the Obi9 connector. Covers connecting, what the toolset reaches, and how to get task-level guidance once attached."
---

# Obi9 reports connector

Puts the data behind a reader's daily Obi9 reports inside the AI assistant they
already use, and lets them change what arrives. Ask in plain language: the
assistant picks the tool, and the answer carries the same numbers the emails
are built from.

## Connect

Add this address as a custom connector:

```
https://biotech.obi9.ai/mcp-reports
```

In Claude that is Settings, then Connectors, then Add custom connector. Other
assistants word it similarly.

Signing in happens in the browser, not here. The assistant opens an Obi9 page;
choose **Email me a sign-in code**, enter the code that arrives, and approve
the connection. There is no API key to paste and no password to remember.

This connector is the narrower of the two. A reader with a full Obi9 terminal
account wants `obi9-terminal` instead, which reaches companies, filings,
catalysts, valuation and market data as well.

## What the toolset reaches

Three groups, and you do not need to name tools or memorise options.

- **Clinical trials.** US and EU registry records, searched and read, plus how
  a record changed over time and the documents an EU trial has posted.
- **Adverse events.** FDA FAERS activity for a drug, company or watchlist, and
  the individual case reports behind the counts, which is where the actual
  content is.
- **Your reports.** Everything that shapes the emails: which reports arrive,
  which watchlist leads them, what is on the shortlist.

## Get the real guidance from the connector

The connector serves task-level skills of its own, and they are the useful
layer: which tool answers a given question, how to read what comes back, and
what a number does and does not mean. Read the relevant one before improvising
a sequence of calls.

Ask the connector to list its skills, or call its `read_skill` tool. They are
served live, so they stay current in a way this installed file cannot.

## Things worth knowing before you answer

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

## Anything the AI produced is the AI's reading

Analysis text, summaries and significance ratings are produced by a model.
Present them as the AI's judgement of a document, never as an established fact
about the company and never as advice. The underlying document is always linked
and is what settles a disagreement.

## Availability

Not every capability is enabled on every plan. If a credential cannot reach
something, the reply says so; ask Obi9 to enable it rather than working around
it.

Full documentation: https://biotech.obi9.ai/mcp-docs
