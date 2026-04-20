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

Raw entries in `~/.claude.json`:

```json
"obsidian-personal": {
  "type": "stdio",
  "command": "/home/tomkw/.local/bin/mcpvault-wrapper.sh",
  "args": ["/home/tomkw/vaults/LifeOS-vault"],
  "env": {}
},
"obsidian-work": {
  "type": "stdio",
  "command": "/home/tomkw/.local/bin/mcpvault-wrapper.sh",
  "args": ["/home/tomkw/vaults/work-v2-vault"],
  "env": {}
}
```

## Wrapper script

mcpvault uses `#!/usr/bin/env node` — when Claude Code spawns it as a subprocess, nvm has not initialised, so `node` is not in PATH and the process exits immediately (`MCP error -32000: Connection closed`). Using the absolute binary path alone does not fix this.

A wrapper script at `~/.local/bin/mcpvault-wrapper.sh` loads nvm before calling mcpvault:

```bash
#!/usr/bin/env bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
exec mcpvault "$@"
```

The wrapper is Node-version-agnostic — it resolves `mcpvault` via nvm at runtime, so no reregistration is needed after Node upgrades.

## Permissions model

- `--scope user`: registration is in `~/.claude.json`, not in any project's `.claude/` folder
- mcpvault has **full read/write access** to the registered vault directory — no auth layer
- mcpvault deletes are permanent by default — `--trashMode local` flag exists in theory but mcpvault 0.11.0 concatenates all args into the vault path, breaking it; avoid until fixed upstream
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
claude mcp add --scope user obsidian-personal -- ~/.local/bin/mcpvault-wrapper.sh /home/tomkw/vaults/LifeOS-vault
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
