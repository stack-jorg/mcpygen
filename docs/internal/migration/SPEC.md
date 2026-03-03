## Spec: Extract MCP tooling infrastructure to standalone `mcpygen` project

### What
Extract the reusable MCP infrastructure from ipybox (MCPClient, ToolServer, ApprovalChannel/Client, code generation, variable substitution) into a new standalone project at `../mcpygen`. The new project provides a Python library (`import mcpygen`) and CLI tools for MCP API code generation and running a ToolServer. ipybox becomes a consumer of mcpygen.

### Why
The MCP tool server, approval workflow, code generation, and client infrastructure are general-purpose and useful outside the context of Jupyter kernel execution. Extracting them makes this functionality available to any project that wants to manage MCP servers, generate typed Python APIs from MCP tool schemas, or gate tool calls behind an approval workflow.

### Requirements

#### Project setup
- Create `../mcpygen` as a new Python project using `uv init`
- Package name: `mcpygen` (importable as `import mcpygen`)
- Python 3.11+ (matching ipybox)
- Mirror ipybox's project infrastructure:
  - `pyproject.toml` with uv, pytest, pre-commit
  - `tasks.py` with invoke tasks: `cc`, `test`, `ut`, `it`, `build-docs`, `serve-docs`
  - `.pre-commit-config.yaml` matching ipybox's setup
  - `CLAUDE.md` at project root with project-specific guidelines
  - `docs/AGENTS.md` and `tests/AGENTS.md` symlinked as `CLAUDE.md` in their directories
  - Full MkDocs site with Material for MkDocs, mkdocstrings, llms.txt generation
  - `docs/internal/` for architecture and testing docs (not published)
  - Public docs for user-facing API and usage

#### Module structure
- `mcpygen/client.py`: MCPClient (from `ipybox/mcp_client.py`)
- `mcpygen/apigen.py`: code generation (from `ipybox/mcp_apigen.py`)
- `mcpygen/vars.py`: variable replacement utilities (from `ipybox/vars.py`)
- `mcpygen/tool_exec/server.py`: ToolServer (from `ipybox/tool_exec/server.py`)
- `mcpygen/tool_exec/client.py`: ToolRunner, ToolRunnerError (from `ipybox/tool_exec/client.py`)
- `mcpygen/tool_exec/approval/server.py`: ApprovalChannel (from `ipybox/tool_exec/approval/server.py`)
- `mcpygen/tool_exec/approval/client.py`: ApprovalClient, ApprovalRequest (from `ipybox/tool_exec/approval/client.py`)
- `mcpygen/__init__.py`: re-exports of key public API

#### Typed approval errors
- Add `ApprovalRejectedError(ToolRunnerError)` for rejected tool calls
- Add `ApprovalTimeoutError(ToolRunnerError)` for timed-out approval requests
- ToolServer must return distinguishable error responses for rejection vs timeout (not just generic 403/error)
- ToolRunner must parse these responses and raise the appropriate error type
- Both error types should be importable from `mcpygen.tool_exec`

#### CLI entry points
- `mcpygen apigen` -- generate typed Python tool APIs from MCP server schemas (wraps `generate_mcp_sources`)
- `mcpygen toolserver` -- run a standalone ToolServer instance
- Registered as console script entry point in pyproject.toml

#### Tests
- `tests/unit/test_apigen.py`: move unit tests from ipybox (injection safety tests)
- `tests/integration/test_tool_exec.py`: move integration tests from ipybox
- `tests/integration/test_apigen.py`: move integration tests from ipybox
- `tests/integration/mcp_server.py`: move test MCP server (tool-1, tool_2, tool_3) to mcpygen's test suite
- `tests/integration/conftest.py`: move relevant fixtures (stdio/http/sse server params)
- Add tests for `ApprovalRejectedError` and `ApprovalTimeoutError` (verify correct error type is raised on rejection and timeout)

#### Documentation
- `docs/index.md`: project overview, what mcpygen provides
- `docs/installation.md`: installation instructions
- `docs/quickstart.md`: getting started guide
- `docs/client.md`: MCPClient usage
- `docs/apigen.md`: code generation (adapted from ipybox's `docs/apigen.md`)
- `docs/toolserver.md`: ToolServer and approval workflow
- `docs/cli.md`: CLI reference
- `docs/internal/architecture.md`: module structure and dependencies
- `docs/internal/testing.md`: testing conventions

#### ipybox changes
- Add `mcpygen` as a dependency in ipybox's `pyproject.toml` (path dependency during development)
- Replace all imports from `ipybox.mcp_client`, `ipybox.mcp_apigen`, `ipybox.vars`, `ipybox.tool_exec.*` with imports from `mcpygen`
- Remove extracted source files from ipybox
- Remove extracted tests from ipybox (ipybox keeps `test_mcp_server.py` since IpyboxMCPServer stays)
- Update ipybox's `__init__.py` re-exports if any of the moved symbols were public
- Update ipybox's docs to reference mcpygen where appropriate
- Update ipybox's `docs/internal/architecture.md` and `docs/internal/testing.md`

### Out of Scope
- IpyboxMCPServer (`ipybox/mcp_server.py`) -- ipybox-specific orchestration, stays in ipybox
- KernelGateway, KernelClient, CodeExecutor -- ipybox core, not related to MCP infrastructure
- Publishing mcpygen to PyPI -- can happen later
- GitHub Actions / CI setup -- can be added separately

### Key Decisions
- Package name is `mcpygen` (short, matches repo)
- IpyboxMCPServer stays in ipybox; only reusable infrastructure moves
- Test MCP server (tool-1, tool_2, tool_3) moves to mcpygen's test suite; ipybox creates its own test fixtures if needed
- Full MkDocs documentation site from day one
- Two new error types (`ApprovalRejectedError`, `ApprovalTimeoutError`) added during extraction to improve approval error handling
- CLI provides both `apigen` and `toolserver` subcommands

### Constraints
- mcpygen must have zero dependency on ipybox
- ipybox's existing tests that use the moved modules (`test_mcp_server.py`) must still pass after switching to mcpygen imports
- The approval error types require changes on both server side (ToolServer returning typed errors) and client side (ToolRunner raising typed exceptions)
- All function parameters and return types must have type hints
- Modern union syntax (`str | None`), `match`/`case` over `isinstance()`
- No functions or classes defined in `__init__.py` (re-exports only)
