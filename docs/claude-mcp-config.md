# Claude Code MCP Configuration

Documents the MCP (Model Context Protocol) server configuration on this machine — what's registered, where it lives, and how to manage it.

## Config location

```
~/.claude.json
```

User-scoped — applies globally across all Claude Code sessions and projects on this machine. Not tied to any repo or workspace.

## Registered MCP servers

| Name | Vault | Path |
|------|-------|------|
| `obsidian-personal` | LifeOS-vault (personal) | `/home/tomkw/vaults/LifeOS-vault` |
| `obsidian-work` | work-v2-vault | `/home/tomkw/vaults/work-v2-vault` |

Both use `--trashMode local`: deletes go to a `.trash` folder inside the vault rather than being permanently removed.

Raw entries in `~/.claude.json`:

```json
"obsidian-personal": {
  "type": "stdio",
  "command": "/home/tomkw/.nvm/versions/node/v22.22.2/bin/mcpvault",
  "args": ["/home/tomkw/vaults/LifeOS-vault", "--trashMode", "local"],
  "env": {}
},
"obsidian-work": {
  "type": "stdio",
  "command": "/home/tomkw/.nvm/versions/node/v22.22.2/bin/mcpvault",
  "args": ["/home/tomkw/vaults/work-v2-vault", "--trashMode", "local"],
  "env": {}
}
```

## Binary location

mcpvault is installed globally via nvm:

```
/home/tomkw/.nvm/versions/node/v22.22.2/bin/mcpvault
```

The absolute path is used in the MCP registration because nvm is not in PATH for non-interactive shells. If Node is upgraded, re-register with the new path (`which mcpvault` after activating nvm).

## Permissions model

- `--scope user`: registration is in `~/.claude.json`, not in any project's `.claude/` folder
- mcpvault has **full read/write access** to the registered vault directory — no auth layer
- `--trashMode local` is the only delete safeguard; no other access controls exist
- Trust model: mcpvault trusts its parent process (Claude Code); do not register untrusted vaults

## Available tools

Once connected, Claude Code exposes these tool prefixes:

- `mcp__obsidian-personal__*` — personal vault operations
- `mcp__obsidian-work__*` — work vault operations

Key tools: `search_notes`, `read_note`, `write_note`, `list_notes`, `delete_note`, `move_note`.

## Managing servers

```bash
# List registered servers and health status
claude mcp list

# Remove a server
claude mcp remove obsidian-personal

# Re-register (e.g. after Node upgrade)
claude mcp add --scope user obsidian-personal -- /path/to/mcpvault /home/tomkw/vaults/LifeOS-vault --trashMode local
```

## Dependencies

| Dependency | Version | Install method |
|------------|---------|----------------|
| Node.js | v22.22.2 | nvm |
| mcpvault | 0.11.0 | npm install -g |
| nvm | v0.40.3 | curl install script |

## Related

- [env.ubuntu.json](../config/env.ubuntu.json) — vault paths for this platform
- Work vault runbook: `work-v2-vault/3 resource/runbooks/obsidian-mcp-setup.md`
