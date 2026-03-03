# Quickstart

## Generate a typed tool API

```python
from pathlib import Path

from mcpy import generate_mcp_sources

server_params = {
    "command": "uvx",
    "args": ["mcp-server-fetch"],
}

await generate_mcp_sources("fetch_mcp", server_params, Path("mcptools"))
```

This generates a Python package under `mcptools/fetch_mcp/` with one module per tool, each containing a `Params` class and a `run()` function.

## Execute a tool

```python
from mcpy import ToolRunner

runner = ToolRunner(
    server_name="fetch_mcp",
    server_params={"command": "uvx", "args": ["mcp-server-fetch"]},
)

result = await runner.run("fetch", {"url": "https://example.com"})
```

## Run a ToolServer

```python
from mcpy import ToolServer

async with ToolServer(port=8900) as server:
    await server.join()
```

Or from the command line:

```bash
mcpy toolserver --port 8900
```
