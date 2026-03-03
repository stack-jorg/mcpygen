# CLI Reference

mcpy provides a command-line interface with two subcommands.

## `mcpy apigen`

Generate typed Python tool APIs from MCP server schemas.

```bash
mcpy apigen \
    --server-name fetch_mcp \
    --server-params '{"command": "uvx", "args": ["mcp-server-fetch"]}' \
    --root-dir mcptools
```

**Arguments:**

| Argument | Description |
|---|---|
| `--server-name` | Name for the generated package directory |
| `--server-params` | MCP server connection parameters (JSON string) |
| `--root-dir` | Parent directory where the package will be created |

## `mcpy toolserver`

Run a standalone ToolServer instance.

```bash
mcpy toolserver --host localhost --port 8900 --log-level INFO
```

**Arguments:**

| Argument | Default | Description |
|---|---|---|
| `--host` | `localhost` | Hostname to bind to |
| `--port` | `8900` | Port to listen on |
| `--log-level` | `INFO` | Logging level |
