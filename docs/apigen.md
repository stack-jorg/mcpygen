# Python tool API generation

[`generate_mcp_sources()`][mcpy.apigen.generate_mcp_sources] generates a typed Python tool API from MCP server tool schemas. Each tool becomes a module with a Pydantic `Params` class, a `Result` class or `str` return type, and a `run()` function.

## Stdio servers

For MCP servers that run as local processes, specify `command`, `args`, and optional `env`:

```python
from pathlib import Path

from mcpy import generate_mcp_sources

server_params = {
    "command": "npx",
    "args": ["-y", "@anthropic/brave-search-mcp-server"],
    "env": {"BRAVE_API_KEY": "${BRAVE_API_KEY}"},
}

await generate_mcp_sources("brave_search", server_params, Path("mcptools"))
```

## HTTP servers

For remote MCP servers over HTTP, specify `url` and optional `headers`:

```python
server_params = {
    "url": "https://api.github.com/mcp/",
    "headers": {"Authorization": "Bearer ${GITHUB_API_KEY}"},
}

await generate_mcp_sources("github", server_params, Path("mcptools"))
```

mcpy auto-detects the transport type from the URL. URLs containing `/mcp` use streamable HTTP, URLs containing `/sse` use SSE. You can also set `type` explicitly to `"streamable_http"` or `"sse"`.

## Environment variable substitution

You can use `${VAR_NAME}` placeholders in `server_params` values. mcpy replaces them with the corresponding environment variable when connecting to the MCP server.

## Generated package structure

The Brave Search MCP server example above generates a package structure like this:

```
mcptools/
└── brave_search/
    ├── __init__.py
    ├── brave_web_search.py
    ├── brave_local_search.py
    ├── brave_image_search.py
    └── ...
```

For each MCP server tool, a separate Python module is generated, named after the tool.

## Using the generated API

Each module provides a typed interface for programmatic MCP tool calls:

```python
from mcptools.brave_search.brave_image_search import Params, Result, run

# Params validates input
params = Params(query="neural topic models", count=3)

# run() calls the MCP tool and returns a Result (or str for untyped tools)
result: Result = run(params)

for image in result.items:
    print(image.title)
```

The `Params` class is generated from the tool's input schema. Tools with an output schema get a typed `Result` class; others return `str`. The MCP tool itself is called via its `run()` function.

## API Reference

::: mcpy.apigen.generate_mcp_sources
