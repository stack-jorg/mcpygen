# mcpy

mcpy is a Python library for MCP tooling infrastructure. It provides a generic MCP client, typed Python API code generation from MCP server schemas, an HTTP tool server with approval workflow, and CLI tools.

## Features

- **MCPClient**: Connect to MCP servers over stdio, SSE, or streamable HTTP
- **Code generation**: Generate typed Python tool APIs with Pydantic models from MCP server schemas
- **ToolServer**: HTTP server that manages MCP servers and executes tools with optional approval
- **Approval workflow**: WebSocket-based approval channel for gating tool calls
- **CLI**: Command-line tools for code generation and running a ToolServer
