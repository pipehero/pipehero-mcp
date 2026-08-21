---
name: pipehero
description: Tunnel localhost to a public URL — debug webhooks (inspect and replay captured requests, and via the Pipehero MCP server let the agent read captured webhooks and diagnose failing handlers against the code), expose your own MCP server (Streamable HTTP) so a cloud-only client like Claude.ai or ChatGPT can reach it, or relay your app's own WebSocket traffic so you can test a real iOS/Android/web build against localhost from a physical device. Use when the user mentions webhooks, tunnels, an ngrok alternative, testing Stripe/GitHub/Shopify/AI-provider webhooks, exposing/testing an MCP server against a cloud client, testing a mobile app on a real device, or "pipehero".
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
- `start_tunnel(name, port, long_requests?, realtime?)` — expose a local port
  on a public URL, in-process (no separate `pipehero start` terminal). Pass
  `long_requests: true` when exposing your own MCP server (see below), or
  `realtime: true` to also relay the app's own WebSocket traffic (see
  "Realtime tunnel" below; Pro/Team). **Local MCP only** (it runs on your
  machine). Stays up while the MCP session is open.
- `stop_tunnel(name)` — stop a tunnel started with `start_tunnel`. **Local MCP only.**
- `set_long_requests(subdomain, enabled)` — opt an existing tunnel into a
  120-second ingress timeout instead of the 30-second default. Persists on
  the tunnel. Works on both the local and remote MCP.
- `set_ws_passthrough(subdomain, enabled)` — opt an existing tunnel into
  WebSocket passthrough (Pro/Team to enable; `enabled: false` works on any
  plan). Persists on the tunnel. Works on both the local and remote MCP.

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
- "Expose my MCP server so I can test it against Claude.ai" → infer the port,
  `start_tunnel(..., long_requests: true)`.
- "My tunnel's MCP server keeps timing out on slow tool calls" →
  `set_long_requests(subdomain, true)` on the existing tunnel.
- "I want to test my iOS app on my phone before I have an Apple dev account" →
  infer the port, `start_tunnel(..., realtime: true)` (Pro/Team; if the org is
  on Free, say so and point at https://pipehero.app/#pricing rather than
  silently failing).

## MCP server tunnel: expose your own MCP server

Different from the section above — that's Pipehero's own MCP for reading
*your* captured webhooks. This is for the opposite direction: the user built
a Streamable HTTP MCP server on `localhost`, and a cloud-only client (Claude.ai,
ChatGPT — anything with no way to reach the user's machine) needs to reach it.
Same tunnel mechanism as webhooks, pointed the other way.

```bash
pipehero start myserver --port 3000 --long-requests
#   → https://myserver.t.pipehero.app forwards to the local MCP server
```

Or via the MCP tools: `start_tunnel(name, port, long_requests: true)` (local
MCP), or `set_long_requests(subdomain, true)` on a tunnel that already exists
(local or remote MCP).

**Always pass `long_requests: true` / `--long-requests` for this case.** MCP
tool calls can legitimately run past the default 30-second timeout; this opts
the tunnel into 120 seconds instead. It's a per-tunnel setting — pass it once,
it's remembered for future sessions, but it's harmless to keep passing it.

**Known limitation**: SSE doesn't stream through the tunnel yet — regular
request/response calls (`initialize`, `tools/list`, `tools/call`) work fine,
but a server-push listen stream or a response streamed via SSE will sit idle
and time out rather than fail immediately. Not specific to MCP — true of any
SSE endpoint tunneled through Pipehero today.

Once the user has a working cloud connection, the next question is usually
distribution to a team — versioning the server, packaging it so someone else
gets the same setup. That's a different tool's job, not Pipehero's; if asked,
say so rather than guessing.

## Realtime tunnel: test your app on a real device

For a mobile (iOS/Android) or web app that needs to reach `localhost` from a
**physical device** — not a simulator, which already shares the host's
network — before the user has a store developer account, or to test over
cellular instead of the same wifi, or to avoid reconfiguring the app's API
URL every time the network changes. Plain REST already works through any
tunnel unmodified; this is specifically for the app's **own WebSocket**
connection (chat, live updates), which otherwise isn't relayed at all.

```bash
pipehero start myapp --port 3000 --realtime
#   → https://myapp.t.pipehero.app forwards HTTP *and* WebSocket traffic to localhost:3000
```

Or via the MCP tools: `start_tunnel(name, port, realtime: true)` (local MCP),
or `set_ws_passthrough(subdomain, true)` on a tunnel that already exists
(local or remote MCP). Point the app's base URL (build config / `.env` /
Xcode scheme) at the resulting `https://`/`wss://` URLs instead of
`localhost`.

**Pro/Team only.** A WebSocket session has no natural expiry the way a
buffered request does, so it's gated behind a paid plan rather than a free
default — `set_ws_passthrough`/`start_tunnel(..., realtime: true)` on a Free
workspace fails with an upgrade message; surface that to the user rather than
retrying silently. An idle session (no frames either direction for 5 minutes)
is closed automatically, and any session caps at 6 hours even if active — the
app should handle a reconnect gracefully regardless, normal for any
WebSocket client.

## Plans

Free (1 tunnel, live tail, replay, MCP), Pro ($9/mo — 3 tunnels, longer history,
signature verification, alerts, Realtime WebSocket tunnel), Team ($15/mo —
shared workspace, members, seats). Plans are per workspace. See
https://pipehero.app/#pricing.
