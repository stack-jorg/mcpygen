# Implementation Plan

Based on the spec at `../ipybox/SPEC.md`.

## Phase 1: Create mcpy project

### 1.1 Project initialization
- `uv init` at `../mcpy`
- `pyproject.toml` with dependencies: aiofiles, aiohttp, datamodel-code-generator, fastapi, mcp, requests, uvicorn, websockets, wsproto
- Python 3.11+, Apache-2.0 license
- hatchling build system with uv-dynamic-versioning

### 1.2 Extract source modules
File mappings (ipybox -> mcpy):
- `ipybox/mcp_client.py` -> `mcpy/client.py`
- `ipybox/mcp_apigen.py` -> `mcpy/apigen.py`
- `ipybox/vars.py` -> `mcpy/vars.py`
- `ipybox/tool_exec/server.py` -> `mcpy/tool_exec/server.py`
- `ipybox/tool_exec/client.py` -> `mcpy/tool_exec/client.py`
- `ipybox/tool_exec/approval/server.py` -> `mcpy/tool_exec/approval/server.py`
- `ipybox/tool_exec/approval/client.py` -> `mcpy/tool_exec/approval/client.py`

All internal imports updated from `ipybox.*` to `mcpy.*`.

### 1.3 Add typed approval errors
- `ApprovalRejectedError(ToolRunnerError)` in `mcpy/tool_exec/client.py`
- `ApprovalTimeoutError(ToolRunnerError)` in `mcpy/tool_exec/client.py`
- ToolServer returns `{"error": "...", "type": "rejected"|"timeout"}` in error responses
- ToolRunner parses `type` field via `_make_error()` using `match`/`case`

### 1.4 Add CLI entry point
- `mcpy/cli.py` with `apigen` and `toolserver` subcommands
- Registered as `mcpy = "mcpy.cli:main"` console script

### 1.5 Add `__init__.py` re-exports
- `mcpy/__init__.py`: MCPClient, generate_mcp_sources, ToolServer, ToolRunner, ToolRunnerError, ApprovalRejectedError, ApprovalTimeoutError, ApprovalClient, ApprovalRequest
- `mcpy/tool_exec/__init__.py`: ToolRunner, ToolRunnerError, ApprovalRejectedError, ApprovalTimeoutError
- `mcpy/utils.py`: `arun()` helper

### 1.6 Extract tests
- `tests/unit/test_apigen.py`: injection safety tests
- `tests/unit/test_replace_variables.py`: variable replacement tests
- `tests/unit/test_approval_request.py`: ApprovalRequest unit tests
- `tests/integration/mcp_server.py`: test MCP server (tool-1, tool_2, tool_3)
- `tests/integration/conftest.py`: fixtures (ip_address, stdio/http/sse_server_params)
- `tests/integration/test_tool_exec.py`: tool execution tests (with typed error assertions)
- `tests/integration/test_tool_approval.py`: approval workflow tests
- `tests/integration/test_apigen.py`: API generation integration tests
- `__init__.py` in tests/, tests/unit/, tests/integration/

### 1.7 Project infrastructure
- `tasks.py` with invoke tasks: cc, test, ut, it, build-docs, serve-docs
- `.pre-commit-config.yaml` matching ipybox
- `AGENTS.md` at project root, `CLAUDE.md` -> `AGENTS.md` symlink
- `docs/AGENTS.md` + `CLAUDE.md` symlink, `tests/AGENTS.md` + `CLAUDE.md` symlink

### 1.8 Documentation
- `mkdocs.yml` with Material theme, mkdocstrings, llms.txt
- `docs/index.md`, `installation.md`, `quickstart.md`, `client.md`, `apigen.md`, `toolserver.md`, `cli.md`
- `docs/internal/architecture.md`, `docs/internal/testing.md`
- `docs/stylesheets/extra.css`

### 1.9 Verify mcpy tests pass
- All unit tests (42)
- All integration tests (74)

## Phase 2: Update ipybox to depend on mcpy

### 2.1 Add mcpy dependency
- Add `mcpy` to ipybox's `pyproject.toml` dependencies
- Add `[tool.uv.sources]` with path dependency to `../mcpy`
- Remove dependencies now provided transitively by mcpy (aiofiles, aiohttp, datamodel-code-generator, fastapi, mcp, requests, uvicorn, websockets, wsproto)

### 2.2 Update imports in source files
- `ipybox/__init__.py`: re-export `generate_mcp_sources` and `ApprovalRequest` from mcpy
- `ipybox/code_exec.py`: import ApprovalClient, ApprovalRequest, ToolServer, reset from mcpy
- `ipybox/mcp_server.py`: import ToolServer, generate_mcp_sources, reset from mcpy
- Update mkdocstrings cross-references in docstrings

### 2.3 Remove extracted source files
- `ipybox/mcp_client.py`, `ipybox/mcp_apigen.py`, `ipybox/vars.py`
- `ipybox/tool_exec/` directory (server.py, client.py, approval/)

### 2.4 Remove extracted test files
- `tests/unit/test_mcp_apigen.py`, `tests/unit/test_replace_variables.py`, `tests/unit/test_approval_request.py`
- `tests/integration/test_tool_exec.py`, `tests/integration/test_tool_approval.py`, `tests/integration/test_mcp_apigen.py`
- Keep `tests/integration/mcp_server.py` (still used by remaining tests)
- Simplify `tests/integration/conftest.py` (remove unused fixtures)

### 2.5 Update docs
- `docs/api/tool_executor.md`: update mkdocstrings references to mcpy module paths
- `docs/internal/architecture.md`: document mcpy dependency, update module list
- `docs/internal/testing.md`: remove references to moved tests
- Update generated code references (`docs/generated/`)

## Phase 3: Verify ipybox tests pass

- Unit tests (20)
- Integration tests (74): test_code_exec.py, test_kernel_mgr.py, test_mcp_server.py, test_nocolor.py
