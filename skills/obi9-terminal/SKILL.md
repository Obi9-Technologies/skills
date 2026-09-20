---
name: obi9-terminal
description: "Installs and authenticates the Obi9 connector, then uses the full Obi9 biotech toolset: companies and their filings and transcripts, catalysts, clinical trials, FDA adverse events, drug labels, share counts and valuation, market data, and the caller's own watchlists and alerts. WHEN: the user asks you to add, install, connect, fix or re-authenticate the Obi9 MCP connector, or asks a biotech research or valuation question and has an Obi9 account. Covers the install command per client, the browser sign-in, what to do when it fails, and how to get task-level guidance once attached."
---

# Obi9 connector: install, authenticate, use

The Obi9 tools the user already has in the terminal, available inside whichever
AI assistant they work in. Your first job is usually to get the connector
attached. Do that before answering the question they actually asked.

## 1. Install

The address is:

```
https://biotech.obi9.ai/mcp
```

It speaks MCP over Streamable HTTP with OAuth, so any spec-compliant client can
attach. Use the one that matches where you are running.

**Claude Code**

If this skill arrived as the `obi9-terminal` plugin, **the connector is already
registered** and there is nothing to add: the plugin ships it and Claude Code
starts it on enable.

It registers under the SCOPED name, not a bare one:

```
plugin:obi9-terminal:obi9
```

That full string is what every command wants. A bare `obi9` fails, because no
server is called that.

```bash
claude mcp list        # shows it, and health-checks the connection
```

Only if nothing Obi9 appears in `claude mcp list` (someone pasted this file
rather than installing the plugin) add it by hand, and then the name is yours
to choose:

```bash
claude mcp add --transport http obi9 https://biotech.obi9.ai/mcp --scope user
claude mcp login obi9
```

**Claude desktop or claude.ai**

Settings, then Connectors, then Add custom connector, and paste the address.

**Other clients**

Anything that supports a remote MCP server over Streamable HTTP will work.
Point it at the address above as an HTTP (not stdio) server; in editors that
keep an `mcp.json`, that is an entry carrying the `url`. Follow the client's own
documentation for the exact key names, which change more often than this file
can track.

## 2. Authenticate

**Installing does not sign anybody in, and nothing prompts on its own at
install time.** Registering a server and authorising it are separate steps. Do
not report success on the strength of a successful install.

**If you are the agent: you cannot do this step.** Signing in is interactive
and happens in a browser, so it belongs to the human. Your job is to hand off
clearly. Say this:

> Type `/mcp` at the prompt, choose the Obi9 server, and pick Authenticate. A
> browser will open. I cannot run `/mcp` for you, it is a slash command for
> your session, not a shell command.

`/mcp` is the instruction to give, because it works on every version of Claude
Code. There is also a shell equivalent, but only from **v2.1.186** onwards, and
on anything older it fails with `unknown command 'login'`:

```bash
claude --version                                 # 2.1.186 or newer?
claude mcp login plugin:obi9-terminal:obi9       # only if it is
```

If that errors, do not go hunting for another command. Fall back to `/mcp`.

Nothing to create beforehand. Do NOT go looking for a client ID, a client
secret or an API key: the client registers itself with Obi9 automatically and
authentication happens in the browser.

What the user will see, in order:

1. The browser opens on the Obi9 sign-in page.
2. They sign in. Either their normal Obi9 password, or **Email me a sign-in
   code**, which sends a **6-digit** code to their address. The code option
   exists for seats that have no terminal password.
3. They approve the connection for their own account.
4. The browser hands the client its token and the connection is live.

### No browser on this machine

Needs `claude mcp login`, so v2.1.186 or newer. Over SSH, or on Linux with no
display server, it detects the absence of a browser and prints the authorization
URL instead of trying to open one. The user opens that URL on their own machine,
signs in, and pastes the **full redirect URL** from their browser's address bar
back at the prompt. `--no-browser` forces that mode even where a browser
exists.

```bash
claude mcp login plugin:obi9-terminal:obi9 --no-browser
```

The paste step needs an interactive terminal, so an SSH session has to be
attached: connect with `ssh -t`.

This is why Obi9 has no device-code flow with a short code to type. The client
carries the round trip instead.

### Non-interactive runs

In `claude -p` or an SDK run there is no panel and no way to run OAuth. Claude
Code will report the server's tools as unavailable until it is authorised. Sign
in once from an interactive session; the credential persists.
`claude mcp logout <name>` clears it.

**There is no code to copy back into the terminal.** If you are used to GitHub's
device flow, where the CLI shows an eight-character code to type into a web
page, this is not that. Obi9 uses the authorization code flow with PKCE and a
loopback redirect, so the credential returns to the client automatically. The
only code anywhere in this flow is the 6-digit email one, and it is typed into
the browser, not into your terminal.

The token is per user. Everyone who connects acts as themselves, with their own
watchlists and alerts, so never share one.

## 3. When it does not work

- **Tools listed but every call returns unauthorized.** The connector is
  registered but not signed in. Re-run the sign-in (`/mcp` in Claude Code).
- **Sign-in page opens but the client never receives the token.** The redirect
  comes back to a loopback address on the user's machine. A VPN or a proxy that
  intercepts localhost will break it. Retry off the VPN.
- **The client insists on a client ID or secret.** It has not found our
  discovery documents. Confirm the address is exactly the one above, with no
  trailing slash or path, and that the machine can reach `biotech.obi9.ai`.
- **The client needs a fixed callback port.** Claude Code accepts
  `--callback-port <port>` on the add command.
- Ask the user to confirm they have an Obi9 account at all. This connector
  authenticates an existing account; it cannot create one.

## 4. Prove it works before saying it works

A connector that lists its tools is not a connector that works. Run these two
checks and show the user the answers. Both must go through the connector: if
you find yourself answering from memory, the test has failed.

**Check one, does data come back.** Ask the connector for the fully diluted
share count for a liquid name, say ACHV, and report the number WITH the basis
and the date it is as of. A real answer is a specific figure with an as-of
date. A vague one, or a number with no date, means you answered from training
data and not from Obi9.

**Check two, is it bound to the right account.** Ask the connector for the
caller's own watchlists. This is the check that matters, because it can only
be answered by a token tied to a real Obi9 user. A list of names, even an empty
one, is a pass. An authorization error here with a pass on check one means the
connector attached but the sign-in did not complete.

Always name the tool you called. The user needs to be able to tell a real
answer from a plausible sentence, and naming the tool is what lets them.

If either check fails, work through the troubleshooting above rather than
reporting success.

## 5. What the toolset reaches

Orientation, so you know when Obi9 is the right place to look rather than the
open web. You do not need to name tools or memorise options.

- **Companies and documents.** SEC filings and their exhibits, earnings-call
  and conference transcripts, press releases, already parsed and resolved to a
  company.
- **Catalysts.** The forecast calendar of readouts, regulatory decisions and
  milestones, with the history of how each expected date has moved.
- **Clinical trials.** US and EU registry records, what changed on a trial over
  time, the documents an EU trial has posted, and posted results.
- **Adverse events.** FDA FAERS activity and the individual case reports behind
  it.
- **Drug labels.** The current FDA label, with links to the authoritative
  documents.
- **Share counts and valuation.** Fully diluted counts, the dilution breakdown
  and the instruments behind it, market cap and enterprise value.
- **Market data.** Single data points, and the days a stock moved sharply.
- **Your account.** The caller's watchlists, shortlist, alerts and digests,
  readable and editable in conversation.

If the user wants a key-authenticated REST surface for scripts and spreadsheets
instead, that is a different product: see the `obi9-api` skill.

## 6. Get the real guidance from the connector

The connector serves task-level skills of its own, and they are the useful
layer: which tool answers a given question, how to read what comes back, and
the traps in each data family. Read the relevant one before improvising a
sequence of calls.

Ask the connector to list its skills, or call its `read_skill` tool. They are
served live, so they stay current in a way this installed file cannot.

## 7. Before you answer

**Account tools write to a real account.** Watchlist, shortlist and alert tools
change what a person receives by email and what they see in the terminal, and
the effect is immediate. Read the current state before changing it, and say
what you changed.

**Name the basis on any share count or valuation.** Fully diluted and
treasury-stock-method counts answer different questions and can differ
substantially on the same company at the same moment. Anything price-dependent
also needs the price it was computed at.

**Anything the AI produced is the AI's reading.** Analysis text, summaries and
significance ratings come from a model. Present them as the AI's judgement of a
document, never as an established fact about the company and never as advice.
The underlying document is always linked and is what settles a disagreement.

## Availability

Not every capability is enabled on every plan. If a credential cannot reach
something, the reply says so; ask Obi9 to enable it rather than working around
it.

Full documentation: https://biotech.obi9.ai/mcp-full-docs
