# Turtini for Claude Code

A Claude Code plugin that wires the [Turtini MCP server](https://www.npmjs.com/package/@turtini/mcp) into Claude Code, so the IDE agent can read and write your Turtini org's data while it builds against it.

Install once. Mint a key. Then ask Claude things like *"create a contact for Ada at ada@example.com"* or *"list the last five articles"* — the agent uses the MCP tools directly.

## Install

```bash
claude plugin install github.com/Turtini/claude-code-plugin
```

That registers the plugin and the bundled `turtini` MCP server entry. The server starts automatically once the `TURTINI_API_KEY` environment variable is set.

## Set up

Run `/turtini` in any Claude Code session and it will walk you through:

1. **Mint an API key** at https://turtini.com/settings?tab=integrations → Partner Dev → API Keys (pick the scopes you want the agent to have).
2. **Set the env var** in your shell rc:
   ```bash
   echo 'export TURTINI_API_KEY=turtini_xxx' >> ~/.zshrc && source ~/.zshrc
   ```
3. **Restart Claude Code** so the MCP server picks up the env var.
4. **Verify** with `claude mcp list` — `turtini` should show **connected**.

## What you get

| Tool | Purpose |
|------|---------|
| `auth_whoami` | Confirm which Turtini user / org the key belongs to |
| `account_list` | List CRM accounts |
| `contact_list` | List people records |
| `contact_create` | Add a new contact |
| `opportunity_list` | List sales opportunities |
| `article_list` | List published / draft articles |
| `article_create` | Draft a new article |
| `event_list` | List upcoming events |
| `event_create` | Create a new event |

The toolset grows with `@turtini/mcp` releases. Run `npx -y @turtini/mcp --list-tools` for a live snapshot.

## Slash command

- `/turtini` — guided setup, key minting, verification, troubleshooting.

The skill auto-routes based on what you say (`/turtini install`, `/turtini verify`, `/turtini key`, `/turtini help`).

## Security

- Your API key never leaves your machine. The MCP server runs as a local subprocess of Claude Code; the key is read from your environment, not bundled in this plugin or sent to Turtini outside the standard API calls Claude Code makes on your behalf.
- Scope your key to the minimum the agent needs (`contacts:read`, `articles:write`, etc.). The Turtini Settings page lists every available scope.
- To revoke: delete the key in Turtini Settings; the agent loses access within seconds.

## Uninstall

```bash
claude plugin uninstall turtini
```

Then optionally remove the `export TURTINI_API_KEY=...` line from your shell rc.

## Links

- Turtini: https://turtini.com
- `@turtini/mcp` package: https://www.npmjs.com/package/@turtini/mcp
- Claude Code plugins docs: https://code.claude.com/docs/en/plugins.md
- Issues / feedback: https://github.com/Turtini/claude-code-plugin/issues

---

MIT License.
