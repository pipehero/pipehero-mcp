# Pipehero — MCP server + skill

[**Pipehero**](https://pipehero.app) tunnels your `localhost` to a public URL and captures
every webhook, so you can **inspect, replay, and debug** them — including from your AI
agent via MCP.

> This repo is auto-published from Pipehero's source. It holds the public MCP manifest
> and the agent skill. Issues/PRs welcome.

- 🌐 Website: https://pipehero.app
- 📚 Docs: https://pipehero.app/docs

## Connect the MCP server

Let an agent (Claude Code, Cursor, Copilot, Windsurf, Zed, …) list your tunnels, read
captured webhooks, and replay them to localhost.

### Remote — recommended (no install, OAuth)

```bash
claude mcp add --transport http pipehero https://mcp.pipehero.app/mcp
```

Your client opens the browser to sign in and approve — no token to paste.

For CI/headless, pass a token from https://pipehero.app/dashboard/settings:

```bash
claude mcp add --transport http pipehero https://mcp.pipehero.app/mcp \
  --header "Authorization: Bearer <token>"
```

### Local — via the CLI

```bash
curl -fsSL https://pipehero.app/install | sh
pipehero login
claude mcp add pipehero -- pipehero mcp
```

Any MCP client works — add it to `.cursor/mcp.json`, `claude_desktop_config.json`, etc.:

```json
{ "mcpServers": { "pipehero": { "command": "pipehero", "args": ["mcp"] } } }
```

## Tools

| Tool | What it does |
|---|---|
| `list_tunnels` | Your tunnels and whether each is online. |
| `list_requests(subdomain)` | Recent captured webhooks for a tunnel. |
| `get_request(subdomain, id)` | Full request + response (headers + body). |
| `replay_request(subdomain, id)` | Replay a webhook to your localhost. |
| `start_tunnel(name, port)` | Expose a local port on a public URL (local MCP only). |
| `stop_tunnel(name)` | Stop a tunnel started with `start_tunnel` (local MCP only). |

Because your agent has both the captured webhook and your codebase, it can explain *why*
a handler failed — then fix it and replay to confirm.

## Skill

Install the skill so your agent knows *when and how* to use Pipehero:

```bash
pipehero skill install
```

or copy [`SKILL.md`](./SKILL.md) into `~/.claude/skills/pipehero/SKILL.md`.

## License

MIT
