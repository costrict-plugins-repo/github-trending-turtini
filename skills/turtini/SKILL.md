---
name: turtini
description: Wire the Turtini MCP server into Claude Code, mint an API key, verify the connection, and surface the tools the IDE agent gets. Use when the user mentions "Turtini", asks how to "set up Turtini MCP", wants to give their Claude Code agent access to a CRM / ops backend, or needs help getting / rotating their TURTINI_API_KEY. Also use when the user is debugging a failing Turtini MCP connection.
allowed-tools: Bash(claude mcp *), Bash(npx -y @turtini/mcp *), Bash(env | grep -i turtini), Bash(echo *), Read
---

# `/turtini` — Connect Claude Code to your Turtini workspace

Turtini is a unified business OS (CRM, accounting, hospitality, planning, sites, email — see `https://turtini.com`). It ships an MCP server (`@turtini/mcp`) that exposes the Turtini Public API as MCP tools, so Claude Code can read and write your org's data while it builds against it.

## What the user gets

Once wired, Claude Code can call these tools without leaving the editor:

- `auth_whoami` — confirm which org / user the key belongs to
- `account_list`, `contact_list`, `contact_create`
- `opportunity_list`
- `article_list`, `article_create`
- `event_list`, `event_create`

Scopes on the API key control which tools are usable.

## How this skill is shipped

This plugin already bundles the MCP server entry in `plugin.json`. When the user installs the plugin, the MCP server is registered automatically — the only remaining setup is the `TURTINI_API_KEY` environment variable. **Do not** ask the user to run `claude mcp add` if the plugin is installed; the entry is already there.

If the user installed only the SKILL (not the whole plugin) — rare — the manual `claude mcp add` route still works and is documented below.

---

## Routing — figure out what the user wants

Read what the user asked. Pick exactly one:

1. **First-time setup** ("set up Turtini", "/turtini install", "wire up MCP", or any first invocation) → §1
2. **Get / rotate an API key** ("where do I get a key", "rotate my key", "scopes") → §2
3. **Verify the connection** ("verify Turtini", "is it connected", "MCP not working") → §3
4. **What can it do** ("what tools does it have", "show me Turtini tools") → §4
5. **Uninstall / disable** → §5

If the intent is unclear, list options 1–4 in one sentence each and ask which.

---

## §1 — First-time setup

Walk the user through these three steps in order. Stop after each step until you've confirmed completion.

### Step 1 — mint a Turtini API key

Tell the user:

> Open **https://turtini.com/settings?tab=integrations** → **Partner Dev → API Keys** → **Create key**.
>
> Pick the scopes you want Claude Code to have. For a typical "agent helps me build" setup, check:
> - `contacts:read`, `contacts:write`
> - `accounts:read`
> - `opportunities:read`
> - `articles:read`, `articles:write`
> - `events:read`
>
> The plaintext key (`turtini_...`) is shown **once** on creation — copy it before closing the dialog.

Wait for the user to confirm they have a key copied to clipboard.

### Step 2 — set the environment variable

The bundled MCP server expects `TURTINI_API_KEY` in its environment. Show the user the exact line for their shell:

**zsh / bash** (most macOS / Linux):
```bash
echo 'export TURTINI_API_KEY=turtini_xxx' >> ~/.zshrc && source ~/.zshrc
```

**fish**:
```bash
echo 'set -gx TURTINI_API_KEY turtini_xxx' >> ~/.config/fish/config.fish
```

**Windows PowerShell**:
```powershell
[System.Environment]::SetEnvironmentVariable('TURTINI_API_KEY','turtini_xxx','User')
```

Tell them to replace `turtini_xxx` with the actual key. Do **not** echo a real key in chat or commit it anywhere.

### Step 3 — restart Claude Code, then verify

The MCP server only picks up new env vars on a fresh process. Tell the user:

> Quit Claude Code (Cmd-Q / Ctrl-Q) and reopen it. Then come back here and say "verify".

When they return, jump to §3.

---

## §2 — Get / rotate an API key

Direct them to **https://turtini.com/settings?tab=integrations → Partner Dev → API Keys**.

Key lifecycle:
- The plaintext key is shown **once** at creation. Copy it immediately.
- To rotate: create a new key with the same scopes, update `TURTINI_API_KEY` in your shell rc, restart Claude Code, then revoke the old key.
- Revoking a key takes effect within a few seconds and breaks any agent still using it.

If the user already has a key but lost the plaintext, tell them: there is no recovery flow — create a new key and revoke the old one.

---

## §3 — Verify the connection

Run, and read the output back to the user:

```bash
claude mcp list
```

What you're looking for:
- `turtini` listed and showing **connected** → success. Tell the user they're done; they can now ask Claude things like "list my Turtini contacts" or "create a contact for Ada Lovelace at ada@example.com" and the agent will use the MCP tools.
- `turtini` listed but **failed** / **pending** → the env var probably isn't set. Run:
  ```bash
  env | grep -i turtini
  ```
  If `TURTINI_API_KEY` is not in the output, walk the user back through §1 Step 2 and remind them to fully restart Claude Code (not just reload).
- `turtini` not listed at all → the plugin isn't installed. Send them to the README at the top of this repo for install instructions.

If the env var is set and the server still fails, run a quick smoke test:

```bash
TURTINI_API_KEY="$TURTINI_API_KEY" npx -y @turtini/mcp --help
```

If that errors, the API key itself may be revoked or have insufficient scopes — direct them back to §2 to re-mint with the right scopes.

---

## §4 — What can the agent do

Tell the user the current tool surface (kept in sync with `@turtini/mcp` releases — verify against `npx -y @turtini/mcp --list-tools` if they want a live list):

- **Read**: `auth_whoami`, `account_list`, `contact_list`, `opportunity_list`, `article_list`, `event_list`
- **Write**: `contact_create`, `article_create`, `event_create`

Useful prompts to try:
- *"Who am I in Turtini?"* → `auth_whoami`
- *"List the last 5 contacts."* → `contact_list { limit: 5 }`
- *"Add Ada Lovelace as a contact, ada@example.com."* → `contact_create { firstName, lastName, email }`
- *"Show me articles about onboarding."* → `article_list { search: 'onboarding' }`

The agent has the full schema for each tool — it doesn't need you to memorize argument shapes.

---

## §5 — Uninstall / disable

To disable temporarily without uninstalling, set `TURTINI_API_KEY` to an empty string and restart Claude Code — the server will fail to start and Claude Code will continue without the Turtini tools.

To uninstall the plugin entirely:
```bash
claude plugin uninstall turtini
```

Then, optionally, remove the env var:
```bash
unset TURTINI_API_KEY  # current shell only — also delete the export line from your shell rc
```

---

## Manual `claude mcp add` path (only if the plugin isn't installed)

If for some reason the user needs to add the MCP server without the plugin (raw skill install, custom config), the equivalent CLI is:

```bash
claude mcp add turtini -- npx -y @turtini/mcp
claude mcp env turtini TURTINI_API_KEY turtini_xxx
```

Then restart Claude Code and verify per §3.
