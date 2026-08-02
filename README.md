# CCAPI MCP Server

Image, video, music and text generation across 100+ models, over the
[Model Context Protocol](https://modelcontextprotocol.io) — one hosted endpoint,
no local process to run.

**Endpoint:** `https://api.ccapi.ai/mcp` (Streamable HTTP)
**Docs:** <https://ccapi.ai/mcp>
**Registry:** [`ai.ccapi/mcp`](https://registry.modelcontextprotocol.io/v0/servers?search=ai.ccapi) ·
[Glama](https://glama.ai/mcp/connectors/ai.ccapi/mcp)

This repository holds the published server manifest and the client
documentation. The gateway implementation itself is closed source.

## Install

```json
{
  "mcpServers": {
    "ccapi": {
      "type": "http",
      "url": "https://api.ccapi.ai/mcp",
      "headers": { "Authorization": "Bearer sk-YOUR-KEY" }
    }
  }
}
```

Claude Code:

```bash
claude mcp add --transport http ccapi https://api.ccapi.ai/mcp \
  --header "Authorization: Bearer sk-YOUR-KEY"
```

A CCAPI API key is required — the server returns `401` without one. Keys are
issued at <https://ccapi.ai>. `list_models`, `get_balance` and `get_task` do not
consume quota; the generation tools do.

## Tools

Eleven tools across five jobs.

| Tool | What it does |
|---|---|
| `generate_image` | Text to image |
| `edit_image` | Edit an existing image from a prompt |
| `upscale_image` | Increase resolution |
| `generate_video` | Text or image to video, optionally with a soundtrack |
| `generate_music` | Prompt to a full track |
| `generate_lyrics` | Write lyrics |
| `extend_music` | Continue an existing track |
| `chat_completion` | Text generation across the available models |
| `list_models` | Resolve model names and see what is available |
| `get_balance` | Remaining quota |
| `get_task` | Poll an asynchronous job |

Video and music generation are asynchronous: those tools return a task id, and
the assistant polls `get_task` until the job finishes.

## Scoped endpoints

Alongside the full endpoint, four scoped endpoints each carry a single
capability plus the shared discovery and polling tools:

| Endpoint | Tools |
|---|---|
| `https://api.ccapi.ai/mcp` | all eleven |
| `https://api.ccapi.ai/mcp/image` | `generate_image`, `edit_image`, `upscale_image` + shared |
| `https://api.ccapi.ai/mcp/video` | `generate_video` + shared |
| `https://api.ccapi.ai/mcp/music` | `generate_music`, `generate_lyrics`, `extend_music` + shared |
| `https://api.ccapi.ai/mcp/text` | `chat_completion` + shared |

Shared across every scope: `list_models`, `get_balance`, `get_task`.

They exist because tool selection degrades as the list grows — a model choosing
between four tools picks correctly more often than one choosing between eleven.
If an agent only ever generates images, point it at `/mcp/image`.

The registry advertises only `https://api.ccapi.ai/mcp`, since a client should
be offered one server rather than five variants of the same one. The scoped
endpoints are a deliberate choice by the integrator, not a menu.

## Manifest

[`server.json`](./server.json) mirrors what is published to the official MCP
registry as `ai.ccapi/mcp`. Ownership is proved by domain verification against
`https://ccapi.ai/.well-known/mcp-registry-auth`.

## Licence

[MIT](./LICENSE), covering the documentation and manifest in this repository.
Use of the hosted service is governed by the terms at <https://ccapi.ai>.
