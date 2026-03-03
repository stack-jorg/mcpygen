# ToolServer and approval workflow

## ToolServer

[`ToolServer`][mcpy.tool_exec.server.ToolServer] is an HTTP server that manages MCP servers and executes their tools. MCP servers are started on demand and cached for subsequent calls.

```python
from mcpy import ToolServer

async with ToolServer(port=8900) as server:
    await server.join()
```

### Endpoints

- `GET /status`: Health check
- `PUT /reset`: Close all managed MCP servers
- `POST /run`: Execute an MCP tool
- `WS /approval`: WebSocket for approval clients

## ToolRunner

[`ToolRunner`][mcpy.tool_exec.client.ToolRunner] is the client for executing MCP tools on a ToolServer.

```python
from mcpy import ToolRunner

runner = ToolRunner(
    server_name="fetch",
    server_params={"command": "uvx", "args": ["mcp-server-fetch"]},
    port=8900,
)

# Async
result = await runner.run("fetch", {"url": "https://example.com"})

# Sync
result = runner.run_sync("fetch", {"url": "https://example.com"})
```

## Approval workflow

When `approval_required=True`, each tool call requires approval via WebSocket before execution.

```python
from mcpy import ApprovalClient, ApprovalRequest, ToolServer

async def on_approval(request: ApprovalRequest):
    print(f"Tool call: {request}")
    await request.accept()  # or request.reject()

async with ToolServer(approval_required=True) as server:
    async with ApprovalClient(callback=on_approval):
        # Tool calls now require approval
        ...
```

### Typed approval errors

ToolRunner raises specific error types for approval failures:

- [`ApprovalRejectedError`][mcpy.tool_exec.client.ApprovalRejectedError]: The tool call was rejected
- [`ApprovalTimeoutError`][mcpy.tool_exec.client.ApprovalTimeoutError]: The approval request timed out

Both inherit from [`ToolRunnerError`][mcpy.tool_exec.client.ToolRunnerError].

## API Reference

::: mcpy.tool_exec.server.ToolServer

::: mcpy.tool_exec.client.ToolRunner

::: mcpy.tool_exec.approval.client.ApprovalClient

::: mcpy.tool_exec.approval.client.ApprovalRequest
