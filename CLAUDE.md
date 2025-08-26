# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Essential Commands
```bash
# Install dependencies and set up environment
make install      # Sets up venv with Python 3.13, installs deps, configures pre-commit
uv sync          # Sync all workspace packages and dependencies

# Run the CLI tool
uv run dad-jokes-mcp --help     # Run the main CLI tool
uv run dad-jokes-mcp doctor     # Check system configuration
uv run dad-jokes-mcp setup      # Configure the tool

# Code Quality
make check       # Run all checks: lock file, pre-commit, mypy, deptry
uv run ruff check packages/   # Run linter only
uv run ruff format packages/  # Format code
uv run mypy                   # Type checking

# Testing
make test        # Run all tests with coverage
uv run pytest packages/dad-jokes-mcp/tests/ -v  # Run specific package tests
uv run pytest -k test_name     # Run specific test

# Build & Documentation
make build       # Build wheel packages
make docs        # Serve documentation locally (http://localhost:8000)
```

### Pre-commit hooks
Run manually: `uv run pre-commit run --all-files`
Auto-fixes: Formatting, trailing whitespace, JSON prettification

## Architecture Overview

### Monorepo Structure
This is a **uv workspace monorepo** for exploring AWS Bedrock AgentCore capabilities. The workspace configuration allows multiple packages to be developed together while maintaining separate concerns.

**Key architectural decisions:**
- Uses `uv` for fast dependency management instead of pip/poetry
- Workspace members defined in root `pyproject.toml` under `[tool.uv.workspace]`
- Each package in `packages/` is independently versioned and can be published

### Package: dad-jokes-mcp
Located in `packages/dad-jokes-mcp/`, this is a CLI application for dad jokes MCP server experiments.

**Architecture:**
- **CLI Framework**: Uses Typer with Rich for terminal UI
- **Configuration**: Pydantic models + YAML files stored via platformdirs
- **Entry Points**: Two command names registered: `dad-jokes-mcp`, `djmcp`
- **State Management**: Global state pattern for cross-command options (verbose mode)

**Key Components:**
- `cli.py`: Main application with command definitions using Typer decorators
- `config.py`: Configuration management with `Config` and `CacheConfig` Pydantic models
- Platform-aware file locations using `platformdirs` for config/cache storage

### Testing Strategy
- Tests located in `packages/*/tests/` directories
- Uses pytest with coverage reporting
- Test configuration reads from root `pyproject.toml`
- Coverage reports generated in XML format for CI integration

### Type Safety
- Strict MyPy configuration enforced
- All packages use type hints extensively
- Pydantic for runtime validation of configuration data

## Important Configuration Details

### Python Version Requirements
- Root workspace: Python >=3.13,<4.0


### Ruff Configuration
- Line length: 120 characters
- Target version: Python 3.13
- Comprehensive rule set including security (S), complexity (C90), and style checks
- Auto-fix enabled in Makefile commands

### Workspace Dependencies
When adding workspace packages as dependencies:
```toml
[project]
dependencies = ["dad-jokes-mcp"]

[tool.uv.sources]
dad-jokes-mcp = { workspace = true }
```

### Build System
Uses Hatchling with specific configuration for package discovery:
```toml
[tool.hatch.build.targets.wheel]
packages = ["src/dad_jokes_mcp"]
```

## AWS AgentCore Integration Points

This playground is designed to explore:
- Long-running asynchronous tasks with AWS Bedrock AgentCore
- Agent runtime session management
- Background task processing patterns

The `dad-jokes-mcp` CLI will be the main entry point for launching and managing AgentCore experiments with dad jokes MCP functionality.