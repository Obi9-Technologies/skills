# Obi9 Agent Skills

Agent skills that teach an AI assistant how to reach [Obi9](https://biotech.obi9.ai)
biotech data: fully diluted share counts and enterprise value, SEC filings and
earnings transcripts, clinical trials, medical conference abstracts, FDA adverse
events, drug labels, catalysts, and a feed of what just happened.

Install one and your assistant knows how to connect, what it can ask for, and
where to find the current reference. Each skill is a bootstrap: the depth is
fetched live from Obi9 once you are connected, so it stays correct as the
product changes.

## Which one

Pick the skill for the surface your Obi9 credential reaches. If you are not
sure, `obi9-terminal` is the right guess for a terminal account and
`obi9-reports` for someone who receives Obi9 reports by email.

| Skill | Surface | Docs |
| --- | --- | --- |
| [`obi9-api`](skills/obi9-api/SKILL.md) | REST and WebSocket at `api.obi9.ai`, authenticated with an API key | [Developer portal](https://biotech.obi9.ai/developers) |
| [`obi9-terminal`](skills/obi9-terminal/SKILL.md) | The full MCP connector at `biotech.obi9.ai/mcp` | [MCP documentation](https://biotech.obi9.ai/mcp-full-docs) |
| [`obi9-reports`](skills/obi9-reports/SKILL.md) | The reports MCP connector at `biotech.obi9.ai/mcp-reports` | [Reports documentation](https://biotech.obi9.ai/mcp-docs) |

## Install

**`obi9-api` is one file.** Save
[`skills/obi9-api/SKILL.md`](skills/obi9-api/SKILL.md) as `obi9-api/SKILL.md`
in your agent's skills folder (`~/.cursor/skills` in Cursor, `~/.claude/skills`
in Claude Code) and set your key as the `OBI9_API_KEY` environment variable.
There is nothing else to install.

For the two connector skills, use one of these (one method, not both). The
Claude Code plugin also adds the connector itself; elsewhere the skill walks
your agent through adding it.

**Claude Code**

```bash
claude plugin marketplace add Obi9-Technologies/skills
claude plugin install obi9-terminal@obi9
```

Swap `obi9-terminal` for `obi9-api` or `obi9-reports`.

**Any other agent**

```bash
npx skills add Obi9-Technologies/skills --skill obi9-terminal
```

Add `--list` to see what is available, or `-g` to install for every project
rather than just this one. The CLI asks which agent to install into.

**Or just read it.** Each skill is a single markdown file. Paste one into your
assistant and it works, with no installation at all.

## Getting an Obi9 credential

The MCP connectors sign you in from inside your assistant: add the address, and
Obi9 emails you a sign-in code. There is no key to paste.

The REST API uses an API key, which you create in the
[developer portal](https://biotech.obi9.ai/developers). Keep it in the
`OBI9_API_KEY` environment variable rather than pasting it into a chat, and
register the IP address you will call from in the same portal: keys work only
from registered addresses.

Not every capability is enabled on every plan. If your credential cannot reach
something, the response says so.

## A note on AI-generated text

Obi9 uses AI to analyse documents. Analysis text, summaries and significance
ratings are the AI's reading of a source, not established fact and not advice.
Every response links the document it came from, and that document is what
settles a question.

## Feedback

Issues and pull requests are welcome here. For anything about your account or
your data, contact Obi9 directly.
