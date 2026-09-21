---
name: obi9-reports
description: "Installs and authenticates the Obi9 reports connector, then uses it: the clinical-trial and FDA adverse-event data behind the reader's Obi9 emails, plus control of the reports themselves, which watchlist leads them and what is on their shortlist. WHEN: the user asks you to add, install, connect, fix or re-authenticate the Obi9 connector, or receives Obi9 reports and asks about a trial, an adverse-event report, or changing what they receive. Covers the install command per client, the browser sign-in, what to do when it fails, and how to get task-level guidance once attached."
---

# Obi9 reports connector: install, authenticate, use

Puts the data behind a reader's daily Obi9 reports inside the AI assistant they
already use, and lets them change what arrives. Your first job is usually to
get the connector attached. Do that before answering the question they actually
asked.

## 1. Check before you install anything

**Run `claude mcp list` first.** An Obi9 connector may already be there, and
adding a second one is worse than doing nothing: the same tools then appear
twice under different namespaces and cost twice the context.

It can already be present for reasons that have nothing to do with this skill:

- **Inherited from claude.ai.** A connector added in the claude.ai web UI shows
  up in Claude Code too, listed under `claude.ai` with `Config location:
  claude.ai`. If it says connected, you are DONE. Do not add, do not
  re-authenticate, and never offer Clear authentication, which signs the user
  out of a working connection.
- **Shipped by this plugin**, as `plugin:obi9-reports:obi9-reports`.
- **Added by hand** earlier, under whatever name they chose.

If any of them is connected, skip to the verification checks in section 5. Only
continue here when there is no Obi9 server at all.

## 2. Install

The address is:

```
https://biotech.obi9.ai/mcp-reports
```

It speaks MCP over Streamable HTTP with OAuth, so any spec-compliant client can
attach. Use the one that matches where you are running.

**Claude Code**

If this skill arrived as the `obi9-reports` plugin, **the connector is already
registered** and there is nothing to add: the plugin ships it and Claude Code
starts it on enable.

It registers under the SCOPED name, not a bare one:

```
plugin:obi9-reports:obi9-reports
```

That full string is what every command wants. A bare `obi9-reports` fails,
because no server is called that.

```bash
claude mcp list        # shows it, and health-checks the connection
```

Only if nothing Obi9 appears in `claude mcp list` (someone pasted this file
rather than installing the plugin) add it by hand, and then the name is yours
to choose:

```bash
claude mcp add --transport http obi9-reports https://biotech.obi9.ai/mcp-reports --scope user
claude mcp login obi9-reports
```

**Cursor**

Cursor has no add command; the server goes in a JSON file. User level is
`~/.cursor/mcp.json`, project level is `.cursor/mcp.json`, and the two are
merged with the project one winning on a name clash. A `url` and no `command`
is what marks it as remote:

```json
{
  "mcpServers": {
    "obi9-reports": {
      "url": "https://biotech.obi9.ai/mcp-reports"
    }
  }
}
```

Save, restart Cursor, then open Customize and authenticate the server, or run
`cursor-agent mcp login obi9-reports`. `cursor-agent mcp list` shows status.
Cursor registers itself with Obi9 automatically, so there is no client ID to
set.

**Claude (claude.ai or the desktop app)**

1. Settings, then Connectors, then Add custom connector.
2. Name it `Obi9 Technologies` and paste the address. Leave the OAuth fields
   blank, then Add.
3. Claude opens an Obi9 sign-in and Allow page. Approve it.

**ChatGPT**

1. Turn on Developer mode once: Settings, then Plugins, then Developer mode,
   toggle on.
2. Back in Plugins, choose Browse plugins, then the `+` at the top right,
   next to Search Plugins.
3. Name it `Obi9 Technologies`, paste the address, and set Authentication to
   OAuth. Leave any new fields empty.
4. Sign in and Allow on the Obi9 page that opens.

**Grok**

1. Click the `+` in the message box, then Add connector. Or go to
   grok.com/connectors and choose New Connector.
2. Choose Custom.
3. Name it `Obi9 Technologies`, paste the address, then Add connector.
4. Grok opens an Obi9 sign-in and Allow page. Approve it, and the tools are
   available in your chats.

**Other clients**

Anything that supports a remote MCP server over Streamable HTTP will work.
Point it at the address above as an HTTP (not stdio) server. Follow the
client's own documentation for the exact key names, which change more often
than this file can track.

These steps mirror the walkthrough Obi9 shows under Settings, Connections,
How to. Send anyone who wants to click through it there.

**A remote editor session, such as Cursor over SSH**

Authenticate from a client running on the machine that has the browser. In a
remote editor session the MCP client can be running on the remote host while
the browser callback lands on the laptop, and the token then never reaches the
side that needs it. This is a client limitation rather than anything about
Obi9, and it is not worth fighting: connect from the local machine instead.

This is the narrower of the two Obi9 connectors. A reader with a full Obi9
terminal account wants the `obi9-terminal` skill instead, which also reaches
companies, filings, catalysts, valuation and market data.

## 3. Authenticate

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
claude --version                                     # 2.1.186 or newer?
claude mcp login plugin:obi9-reports:obi9-reports    # only if it is
```

If that errors, do not go hunting for another command. Fall back to `/mcp`.

Nothing to create beforehand. Do NOT go looking for a client ID, a client
secret or an API key: the client registers itself with Obi9 automatically and
authentication happens in the browser.

What the user will see, in order:

1. The browser opens on the Obi9 sign-in page.
2. They choose **Email me a sign-in code** and enter the **6-digit** code that
   arrives. Point them at this rather than at the password box: readers who get
   Obi9 by email generally have no password, so the code is not an alternative
   route for them, it is the route. Do not ask them for a password, and do not
   suggest resetting one.
3. They approve the connection for their own account.
4. The browser hands the client its token and the connection is live.

The code goes to the address the reports arrive at. If they are reading Obi9
emails, that is the address to use.

### No browser on this machine

Needs `claude mcp login`, so v2.1.186 or newer. Over SSH, or on Linux with no
display server, it detects the absence of a browser and prints the authorization
URL instead of trying to open one. The user opens that URL on their own machine,
signs in, and pastes the **full redirect URL** from their browser's address bar
back at the prompt. `--no-browser` forces that mode even where a browser
exists.

```bash
claude mcp login plugin:obi9-reports:obi9-reports --no-browser
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

The token is per user, and it acts on that person's own reports, so never share
one.

## 4. When it does not work

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

## 5. Prove it works before saying it works

A connector that lists its tools is not a connector that works. Run these two
checks and show the user the answers. Both must go through the connector: if
you find yourself answering from memory, the test has failed.

**Check one, does data come back.** Ask the connector for recent FDA
adverse-event activity on a well-known drug, say KEYTRUDA, over the last week.
A real answer names counts or specific case reports. A general description of
what the drug is means you answered from training data and not from Obi9.

**Check two, is it bound to the right account.** Ask the connector what reports
the caller receives and which watchlist leads them. This is the check that
matters, because it can only be answered by a token tied to a real Obi9 user.
Any concrete answer, even an empty shortlist, is a pass. An authorization error
here with a pass on check one means the connector attached but the sign-in did
not complete.

Always name the tool you called. The user needs to be able to tell a real
answer from a plausible sentence, and naming the tool is what lets them.

If either check fails, work through the troubleshooting above rather than
reporting success. Do not change anything on the account as part of testing:
these tools send real email.

## 6. What the toolset reaches

Three groups, and you do not need to name tools or memorise options.

- **Clinical trials.** US and EU registry records, searched and read, plus how a
  record changed over time and the documents an EU trial has posted.
- **Adverse events.** FDA FAERS activity for a drug, company or watchlist, and
  the individual case reports behind the counts, which is where the actual
  content is.
- **Your reports.** Everything that shapes the emails: which reports arrive,
  which watchlist leads them, what is on the shortlist.

## 7. Get the real guidance from the connector

The connector serves task-level skills of its own, and they are the useful
layer: which tool answers a given question, how to read what comes back, and
what a number does and does not mean. Read the relevant one before improvising
a sequence of calls.

Ask the connector to list its skills, or call its `read_skill` tool. They are
served live, so they stay current in a way this installed file cannot.

## 8. Before you answer

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
