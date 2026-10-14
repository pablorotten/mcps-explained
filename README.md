# mcps-explained

A plain-language explanation of MCP (Model Context Protocol). No hype, no jargon.

## What is an MCP?

**MCP is an open standard for connecting an AI to external tools.**

Written by Anthropic (Claude's maker), free and public. Anyone can implement it — which is why
Claude, opencode, Cursor and VS Code all talk to the same servers.

Two things to hold onto:

1. **The protocol is fixed by the spec.** Every MCP server speaks the same three methods.
2. **The tools are not.** Names, arguments and behaviour are yours to design.

Same protocol, wildly different jobs. A docs-search server, a GitHub server that opens issues, a
Postgres server that runs queries — all identical on the wire.

### Where it's defined

<https://modelcontextprotocol.io/specification>

The spec covers:

- The envelope: `jsonrpc`, `id`, `method`, `params`
- The three methods: `initialize`, `tools/list`, `tools/call`
- The transports: HTTP (remote) or stdio (local)

It does **not** cover your tool names or arguments.

## Is it a database? Is another AI listening?

Usually neither.

An MCP server is just a small program that exposes a list of actions. There's no database and no
model behind it. When you search something, the words go straight through — no AI interprets them.
The only AI involved is the one running on your machine.

## Is it like SQL?

The grammar is standardized the same way, but nothing else is.

|          | SQL                     | MCP                             |
| -------- | ----------------------- | ------------------------------- |
| Standard | `SELECT`, `WHERE`, ...  | `tools/list`, `tools/call`      |
| Per-thing| your table names        | your tool names and arguments   |

But there's a crucial difference: **with SQL you go read the schema yourself.** With MCP the
schema arrives at your door. `tools/list` returns every tool with its arguments and which ones are
required. You don't reverse-engineer it — you're handed it, every session.

## The best analogy: HTTP

HTTP is a total standard. `GET`, `POST`, headers, `404`. Every site obeys it.

And yet Amazon, Twitter and your bank's login page are completely different inside — different
data, different APIs, wildly different quality.

HTTP's standardness is exactly what lets a browser talk to all of them without knowing anything
in advance.

MCP is that, for AI tools.

## What actually happens

### At startup (automatic — you do nothing)

```
opencode  →  POST /mcp    initialize     "what are you?"
mcp       →  "I'm server-x v1.2.3, I support tools"
opencode  →  POST /mcp    tools/list     "list them"
mcp       →  2 tools + their argument schemas
```

opencode writes those schemas into the model's context. That's the whole "the AI understands it"
part.

### When you ask a question

```
LLM       →  decides to call a tool, writes the arguments
opencode  →  POST /mcp    tools/call     {name, arguments}
mcp       →  forwards to whatever it wraps, returns text
opencode  →  puts that text in the conversation
LLM       →  reads it, answers you
```

The server sends you the form. The model fills it in. Same as any API.

## Why bother, versus just giving it the files?

Because of what you *don't* read.

Given raw files, the AI has to fetch everything and find the relevant part itself. Given a search
tool, it asks which parts match and gets back a handful.

Caveat worth stating plainly: **this only pays off when the corpus is large.** For a documentation
site with 17 sections, searching saves almost nothing. The docs for this repo's example claim
otherwise. The mechanism is real; the benefit is proportional to how much there is to search.

## Try it yourself

You don't need an AI to talk to an MCP server. `curl` is enough.

```bash
curl -X POST 'https://docs-mcp.monei.com/mcp' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/list","params":{}}'
```

Two gotchas that will cost you an hour if nobody tells you:

1. **`Accept` is one header, comma-separated** — `application/json, text/event-stream`. Two
   separate `Accept` headers returns `406`.
2. **You must send a body.** Empty body returns `400` / `-32600`.

### What the errors teach you

The server will usually tell you what's wrong:

| Error                          | What it means                    |
| ------------------------------ | -------------------------------- |
| `"Invalid JSON payload"`       | your body isn't JSON             |
| `"Client must accept both..."` | you split the `Accept` header    |
| `"Method not found"`           | wrong method name — check the spec |

But it will **not** tell you the valid method names. That part you get from the spec.

## Connecting one to opencode

```jsonc
// ~/.config/opencode/opencode.jsonc
{
  "mcp": {
    "monei": {
      "type": "remote",
      "url": "https://docs-mcp.monei.com/mcp",
      "enabled": true
    }
  }
}
```

Restart. Check with `opencode mcp list`. The tools are now available.

Or from the terminal:

```bash
opencode mcp add monei --url https://docs-mcp.monei.com/mcp
```

`"type": "remote"` means it runs on someone else's server — you give a URL. `"local"` would mean
a command to run on your machine.

## Misconceptions, corrected

**"It's a black box, you have to figure it out one by one."**
No. `tools/list` hands you the menu every session.

**"The protocol is standardized, but competence isn't."**
Correct, and worth internalizing. Same three messages can expose a read-only doc search or an
unattended SQL runner. The standard makes servers *connectable*, not *trustworthy*. Read the tools
before you enable them.

**"It's just documents."**
Docs are the boring read-only case. The same protocol creates issues, sends messages, writes files.
The standard doesn't restrict you to reading.

**"It's new."**
The pattern is HTTP in 1995. It's a small, stable envelope that lets one client talk to many
differing servers.

## Privacy note

MCP servers decide what they send to whom. A real example: Algolia's docs-search server requires
every call to include `userIntent` and `originalQuery` — your verbatim prompt plus the model's
reasoning about it. Not optional; the server rejects calls without them.

Before enabling anything, check what data it requires and where it goes.

## Real example: MONEI

The [`monei` MCP server](https://docs-mcp.monei.com/mcp) is Algolia's generic MCP server pointed at
MONEI's documentation. Two tools: search the docs, and list values in a field.

It's read-only and touches no account, payment or customer data.

It also demonstrates the whole point: the server's own `initialize` response identifies it as
`algolia-mcp v1.22.2` — not custom MONEI code, a wrapper around the same thing that powers their
website search box.

## Sources

- [MCP specification](https://modelcontextprotocol.io/specification)
- [MONEI docs for AI agents](https://docs.monei.com/ai-agents/)
- [opencode MCP docs](https://opencode.ai/docs/mcp-servers/)
