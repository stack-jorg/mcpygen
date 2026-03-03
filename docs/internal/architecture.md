# Architecture

This page documents mcpy internal architecture for agent consumption.

## Key Modules

- `mcpy/client.py`: `MCPClient` - generic MCP client (stdio, SSE, streamable HTTP)
- `mcpy/apigen.py`: `generate_mcp_sources()` - generates typed Python wrappers from MCP schemas
- `mcpy/vars.py`: `replace_variables()` - variable substitution in config dicts
- `mcpy/tool_exec/server.py`: `ToolServer` - FastAPI server managing MCP servers and tool calls
- `mcpy/tool_exec/client.py`: `ToolRunner` - client for executing MCP tools on ToolServer
- `mcpy/tool_exec/client.py`: `ApprovalRejectedError`, `ApprovalTimeoutError` - typed approval errors
- `mcpy/tool_exec/approval/server.py`: `ApprovalChannel` - server-side approval request workflow
- `mcpy/tool_exec/approval/client.py`: `ApprovalClient` - client-side approval handling
- `mcpy/cli.py`: CLI entry point (`apigen`, `toolserver` subcommands)

## Execution Flow

1. User code calls a generated MCP wrapper function
2. Wrapper -> `ToolRunner.run_sync()` -> HTTP POST to ToolServer `/run`
3. ToolServer -> `ApprovalChannel.request()` -> WebSocket -> `ApprovalClient`
4. Application callback receives `ApprovalRequest`, calls `accept()`/`reject()`
5. If accepted: ToolServer executes the MCP tool on the MCP server
6. Result returned by generated wrapper function

## Code Generation

`generate_mcp_sources()` connects to an MCP server, discovers tools, and generates:
- One module per tool with `Params` (Pydantic), optional `Result`, and `run()` function
- `__init__.py` with ToolRunner setup
- Uses `datamodel-code-generator` for schema -> Pydantic conversion

## Approval Error Types

ToolServer returns typed error responses:
- `{"error": "...", "type": "rejected"}` for rejected approvals
- `{"error": "...", "type": "timeout"}` for timed-out approvals
- `{"error": "..."}` (no type) for other errors

ToolRunner maps these to:
- `ApprovalRejectedError` for rejection
- `ApprovalTimeoutError` for timeout
- `ToolRunnerError` for other errors
