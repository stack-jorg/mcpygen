# MCPClient

[`MCPClient`][mcpy.client.MCPClient] connects to MCP servers and provides methods to list tools and execute them.

## Transport types

MCPClient supports three transport types:

- **stdio**: Local process with `command` and `args`
- **Streamable HTTP**: Remote server with `url` containing `/mcp`
- **SSE**: Remote server with `url` containing `/sse`

The transport is auto-detected from the URL or can be set explicitly with `type`.

## Usage

```python
from mcpy import MCPClient

# stdio transport
async with MCPClient({"command": "uvx", "args": ["mcp-server-fetch"]}) as client:
    tools = await client.list_tools()
    result = await client.run("fetch", {"url": "https://example.com"})
```

```python
# HTTP transport
async with MCPClient({"url": "https://api.example.com/mcp"}) as client:
    tools = await client.list_tools()
```

## Environment variable substitution

`MCPClient` replaces `${VAR_NAME}` placeholders in server params with environment variables at connection time.

## API Reference

::: mcpy.client.MCPClient
