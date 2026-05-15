# CLI Reference

This document covers the `rpgkit` command-line interface. Use the CLI to install templates, initialize projects, update RPG-Kit files, and verify local tool availability.

## `rpgkit init`

Initialize a new project from the latest template, or add RPG-Kit to an existing repository.

```bash
rpgkit init <project-name> [options]
rpgkit init --here [options]
rpgkit init . [options]
```

### Options

| Option | Description |
| ------ | ----------- |
| `--ai <agent>` | AI assistant: `copilot` or `claude` |
| `--script <type>` | Script type: `sh` (POSIX) or `ps` (PowerShell) |
| `--here` | Initialize in current directory |
| `--force` | Skip confirmation for non-empty current directory |
| `--no-git` | Skip git initialization |
| `--no-mcp` | Skip MCP server configuration |
| `--ignore-agent-tools` | Skip checks for AI agent CLI tools |
| `--github-token <token>` | GitHub token for private repos or higher rate limits |
| `--pre` | Download the latest pre-release template |
| `--skip-tls` | Skip SSL/TLS verification |
| `--encode/--no-encode` | Run or skip initial RPG encoding at the end of init |
| `--debug` | Show verbose diagnostic output |

### Supported AI Assistants

| Agent | Folder | Description | Status |
| ----- | ------ | ----------- | ------ |
| `copilot` | `.github/`, `.vscode/` | GitHub Copilot | Verified |
| `claude` | `.claude/` | Claude Code | Verified |

RPG-Kit currently supports only **GitHub Copilot** and **Claude Code** in the CLI. Additional agents may be adapted in future releases.

### Examples

```bash
rpgkit init my-project
rpgkit init my-project --ai claude --script sh
rpgkit init . --force
rpgkit init . --encode
rpgkit init . --force --encode
rpgkit init --here --ai copilot
rpgkit init --here --github-token $GITHUB_TOKEN
```

## `rpgkit update`

Update RPG-Kit template files, scripts, command definitions, MCP configuration, gitignore rules, and hooks in an existing project. The AI assistant is auto-detected from existing project configuration when possible.

```bash
rpgkit update
rpgkit update --ai claude
rpgkit update --pre
rpgkit update --no-mcp
rpgkit update --github-token $GITHUB_TOKEN
```

### Options

| Option | Description |
| ------ | ----------- |
| `--ai <agent>` | AI assistant, auto-detected if not specified |
| `--script <type>` | Script type: `sh` (POSIX) or `ps` (PowerShell) |
| `--github-token <token>` | GitHub token for private repos or higher rate limits |
| `--pre` | Download the latest pre-release template |
| `--no-mcp` | Skip MCP server configuration |
| `--skip-tls` | Skip SSL/TLS verification |
| `--debug` | Show verbose diagnostic output |

## `rpgkit check`

Verify that required tools are installed.

```bash
rpgkit check
```

Run this after installation to confirm Python, Git, uv, and the selected AI assistant CLI are available.

## `rpgkit version`

Display version and system information.

```bash
rpgkit version
```

## Network and Release Options

```bash
rpgkit init my-project --github-token $GITHUB_TOKEN
rpgkit init my-project --pre
rpgkit init my-project --skip-tls
rpgkit init my-project --debug
```

| Option | Description |
| ------ | ----------- |
| `--github-token <token>` | Uses a GitHub token for API requests, useful for private repos or rate limits |
| `--pre` | Downloads the latest pre-release template instead of the latest stable release |
| `--skip-tls` | Skips SSL/TLS verification; use only for constrained environments |
| `--debug` | Prints verbose diagnostic output for network and extraction failures |

`GH_TOKEN` and `GITHUB_TOKEN` are also recognized for GitHub API requests.
