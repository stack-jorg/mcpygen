# Implementation State

All phases complete. Both mcpygen and ipybox are fully functional.

## mcpygen project

### Source modules

| Module | Origin | Notes |
|--------|--------|-------|
| `mcpygen/__init__.py` | New | Re-exports: MCPClient, generate_mcp_sources, ToolServer, ToolRunner, ToolRunnerError, ApprovalRejectedError, ApprovalTimeoutError, ApprovalClient, ApprovalRequest |
| `mcpygen/client.py` | `ipybox/mcp_client.py` | MCPClient (stdio, SSE, streamable HTTP) |
| `mcpygen/apigen.py` | `ipybox/mcp_apigen.py` | Code generation; generated code imports from `mcpygen.tool_exec.client` |
| `mcpygen/vars.py` | `ipybox/vars.py` | Variable replacement utilities |
| `mcpygen/cli.py` | New | CLI entry point: `mcpygen apigen`, `mcpygen toolserver` |
| `mcpygen/utils.py` | `ipybox/utils.py` | `arun()` async helper (subset) |
| `mcpygen/tool_exec/__init__.py` | New | Re-exports typed errors and ToolRunner |
| `mcpygen/tool_exec/server.py` | `ipybox/tool_exec/server.py` | ToolServer; returns `"type": "rejected"/"timeout"` in error responses |
| `mcpygen/tool_exec/client.py` | `ipybox/tool_exec/client.py` | ToolRunner + new: ApprovalRejectedError, ApprovalTimeoutError, `_make_error()` |
| `mcpygen/tool_exec/approval/server.py` | `ipybox/tool_exec/approval/server.py` | ApprovalChannel |
| `mcpygen/tool_exec/approval/client.py` | `ipybox/tool_exec/approval/client.py` | ApprovalClient, ApprovalRequest |

### Typed approval errors (new feature)

Server side (`tool_exec/server.py`):
- Rejection: `{"error": "...", "type": "rejected"}`
- Timeout: `{"error": "...", "type": "timeout"}`

Client side (`tool_exec/client.py`):
- `_make_error()` uses `match`/`case` on `response_json.get("type")` to raise:
  - `ApprovalRejectedError(ToolRunnerError)` for `"rejected"`
  - `ApprovalTimeoutError(ToolRunnerError)` for `"timeout"`
  - `ToolRunnerError` for anything else

### Tests

- **116 total** (42 unit + 74 integration)
- Unit: `test_apigen.py` (23 tests), `test_replace_variables.py` (17 tests), `test_approval_request.py` (2 tests)
- Integration: `test_tool_exec.py` (50 tests, parametrized async/sync), `test_tool_approval.py` (18 tests), `test_apigen.py` (6 tests)

### Infrastructure

- `pyproject.toml`: uv, hatchling, uv-dynamic-versioning, ruff, mypy
- `override-dependencies`: `sse-starlette==3.0.3` (see known issue below)
- `tasks.py`: invoke tasks (cc, test, ut, it, build-docs, serve-docs)
- `.pre-commit-config.yaml`: ruff check/format
- `mkdocs.yml`: Material theme, mkdocstrings, llms.txt
- `AGENTS.md` + `CLAUDE.md` symlinks at root, docs/, tests/

### Known issue: sse-starlette pin

`sse-starlette>=3.1` (transitive dependency from `mcp`) causes ASGI protocol errors
(`Expected ASGI message 'http.response.body', but got 'http.response.start'`) during
fixture teardown in parametrized integration tests. The `[async]` variant passes but
`[sync]` (which uses `run_in_executor` via `arun`) hangs indefinitely. Pinning
`sse-starlette==3.0.3` via `override-dependencies` resolves this. ipybox's dependency
tree naturally resolves to 3.0.3 and is unaffected.

## ipybox changes

### Dependencies
- Removed direct dependencies now provided by mcpygen: aiofiles, aiohttp, datamodel-code-generator, fastapi, mcp, requests, uvicorn, websockets, wsproto
- Added: `mcpygen` with `[tool.uv.sources]` path dependency to `../mcpygen`

### Import updates

| File | Old imports | New imports |
|------|------------|-------------|
| `ipybox/__init__.py` | `ipybox.mcp_apigen`, `ipybox.tool_exec.approval.client` | `mcpygen` |
| `ipybox/code_exec.py` | `ipybox.tool_exec.*` | `mcpygen`, `mcpygen.tool_exec.client` |
| `ipybox/mcp_server.py` | `ipybox.mcp_apigen`, `ipybox.tool_exec.*` | `mcpygen`, `mcpygen.tool_exec.client` |
| `tests/integration/test_mcp_server.py` | `ipybox.mcp_client` | `mcpygen` |
| `tests/integration/test_code_exec.py` | `ipybox.mcp_apigen` | `mcpygen` |

Docstring cross-references in `code_exec.py` updated from `ipybox.tool_exec.*`/`ipybox.mcp_apigen.*` to `mcpygen.*`.

### Removed files

Source (9 files):
- `ipybox/mcp_client.py`, `ipybox/mcp_apigen.py`, `ipybox/vars.py`
- `ipybox/tool_exec/` directory (server.py, client.py, approval/server.py, approval/client.py, two `__init__.py`)

Tests (6 files):
- `tests/unit/test_mcp_apigen.py`, `tests/unit/test_replace_variables.py`, `tests/unit/test_approval_request.py`
- `tests/integration/test_tool_exec.py`, `tests/integration/test_tool_approval.py`, `tests/integration/test_mcp_apigen.py`

### Kept files
- `tests/integration/mcp_server.py`: still used by `test_code_exec.py` and `test_mcp_server.py`
- `ipybox/utils.py`: `arun()` and `find_free_port()` (ipybox-specific utilities)
- `tests/integration/conftest.py`: emptied (fixtures were only used by removed tests)

### Doc updates
- `docs/api/tool_executor.md`: mkdocstrings references updated to `mcpygen.*` module paths
- `docs/internal/architecture.md`: documents mcpygen as external dependency, lists extracted modules
- `docs/internal/testing.md`: notes that extracted tests moved to mcpygen
- `docs/generated/mcptools/github/__init__.py`: import updated to `mcpygen.tool_exec.client`

### Test results
- **94 total** (20 unit + 74 integration), all passing
