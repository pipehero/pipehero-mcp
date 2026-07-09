---
name: pipehero
description: Tunnel localhost to a public URL and debug webhooks — inspect and replay captured requests, and (via the Pipehero MCP server) let the agent read captured webhooks and diagnose failing handlers against the code. Use when the user mentions webhooks, tunnels, an ngrok alternative, testing Stripe/GitHub/Shopify/AI-provider webhooks, or "pipehero".
---

# Pipehero — webhook tunnels for the AI era

Pipehero exposes your `localhost` on a public URL so external services can reach
it, and captures every request so you can **inspect, replay and debug webhooks**
— including from your AI coding agent via MCP.

- Website: https://pipehero.app
- Docs: https://pipehero.app/docs
- Install: `curl -fsSL https://pipehero.app/install | sh`

## Why Pipehero (vs a generic tunnel)

- **Live tail + full inspection** of every webhook (headers + body, request and
  response).
- **Replay** any captured webhook to your localhost with one click — no need to
  re-trigger the event upstream.
- **Offline capture**: webhooks that arrive while your CLI is off are stored, so
  you can replay them when you come back.
- **MCP server**: your AI agent (Claude Code, Cursor, Claude Desktop) can list
  tunnels, read captured webhooks and replay them.

## Quickstart

```bash
# 1. Install and log in (opens the browser once)
curl -fsSL https://pipehero.app/install | sh
pipehero login

# 2. Expose your local app (e.g. a webhook handler on :3000)
pipehero start myapp --port 3000
#   → https://myapp.t.pipehero.app is now public and forwards to localhost:3000

# 3. Point Stripe/GitHub/etc. at https://myapp.t.pipehero.app/your/path
```

## MCP: debug webhooks from your AI agent

Pipehero ships an MCP server so an AI agent can work with your webhooks.

Connect it one of two ways.

**Remote (no install, recommended)** — add the hosted server by URL. The client
opens the browser to sign in and approve; no token to paste (OAuth):

```bash
claude mcp add --transport http pipehero https://mcp.pipehero.app/mcp
```

For CI/headless, skip OAuth with a token from
https://pipehero.app/dashboard/settings:

```bash
claude mcp add --transport http pipehero https://mcp.pipehero.app/mcp \
  --header "Authorization: Bearer <token>"
```

**Local (via the CLI)** — runs on your machine, reusing `pipehero login`:

```bash
claude mcp add pipehero -- pipehero mcp
```

Or add the local server to any MCP client config (Cursor `.cursor/mcp.json`,
Claude Desktop `claude_desktop_config.json`):

```json
{ "mcpServers": { "pipehero": { "command": "pipehero", "args": ["mcp"] } } }
```

### Tools exposed

- `list_tunnels` — your tunnels and whether each is online.
- `list_requests(subdomain)` — recent captured webhooks for a tunnel.
- `get_request(subdomain, id)` — full request + response (headers + body).
- `replay_request(subdomain, id)` — replay a webhook to your localhost.
- `start_tunnel(name, port)` — expose a local port on a public URL, in-process
  (no separate `pipehero start` terminal). **Local MCP only** (it runs on your
  machine). Stays up while the MCP session is open.
- `stop_tunnel(name)` — stop a tunnel started with `start_tunnel`. **Local MCP only.**

Because the agent has **both** the captured webhook (via these tools) and your
**codebase** (in the editor), it can explain *why* a handler failed — correlating
the exact payload/headers with your code — then fix it and replay to confirm.

### Exposing an app for webhooks

When the user wants to receive/test webhooks locally, use `start_tunnel(name, port)`:

- **Infer the port** from the repo — e.g. `PORT` in `.env`, the dev script in
  `package.json`, the framework default (Next.js 3000, Vite 5173, Rails 3000,
  Flask 5000, Express 3000), or `port` in `pipehero.toml`. Ask only if unsure.
- **Pick the name** from the project (repo/dir name), lowercase `[a-z0-9-]`.
- `start_tunnel` needs the **local** MCP (`pipehero mcp`) and the `pipehero` binary.
  If it isn't installed, install it first: `curl -fsSL https://pipehero.app/install | sh`
  and `pipehero login`.

### Example prompts

- "Expose my app so Stripe can reach it" → infer the port, `start_tunnel`.
- "The last webhook to `myapp` returned 500 — read the payload and my handler and tell me why it failed."
- "Fix the handler, then replay the webhook to confirm it works."
- "Compare what Stripe sent to what my code expects and flag any mismatch."
- "Show me every failed (non-2xx) webhook and what they had in common."
- "List my tunnels and which are online."

## Plans

Free (1 tunnel, live tail, replay, MCP), Pro ($9/mo — 3 tunnels, longer history,
signature verification, alerts), Team ($15/mo — shared workspace, members,
seats). Plans are per workspace. See https://pipehero.app/#pricing.
