# Configuration

This document covers RPG-Kit configuration that is useful after installation: AI assistant setup, MCP registration, auto-approval, hooks, and initial encoding.

## AI Assistant CLI Requirements

RPG-Kit slash commands are executed by an AI coding agent. Before running `rpgkit init`, install and authenticate at least one supported AI assistant CLI.

Currently verified assistants:

| Agent | `--ai` value | Generated configuration | Requirement |
| ----- | ------------ | ----------------------- | ----------- |
| GitHub Copilot | `copilot` | `.github/`, `.vscode/` | Copilot CLI available and authenticated |
| Claude Code | `claude` | `.claude/` | Claude Code CLI available and authenticated |

Use `rpgkit check` to verify required local tools.

```bash
rpgkit check
```

If the selected AI assistant is not found, install and authenticate it, then rerun `rpgkit init` or `rpgkit update`.

## Initialization Options

### AI assistant selection

```bash
rpgkit init my-project --ai claude
rpgkit init my-project --ai copilot
```

If `--ai` is omitted in an interactive terminal, RPG-Kit prompts for a supported assistant.

### Script type

```bash
rpgkit init my-project --script sh
rpgkit init my-project --script ps
```

`sh` installs POSIX shell-oriented command snippets. `ps` installs PowerShell-oriented snippets.

### MCP registration

By default, `rpgkit init` registers the RPG-Kit MCP server for the selected assistant.

```bash
rpgkit init my-project
```

Pass `--no-mcp` to skip MCP registration:

```bash
rpgkit init my-project --no-mcp
rpgkit update --no-mcp
```

Skipping MCP means the slash-command pipeline still works, but the AI assistant will not get the `rpg-tools` graph-query tools automatically.

### Initial encode

The MCP tools query `.rpgkit/data/rpg.json`. For existing codebases, that file is created by the encoder.

`rpgkit init` supports:

```bash
rpgkit init --here --encode
rpgkit init --here --no-encode
```

Behavior:

- `--encode` runs the encoder at the end of init without prompting.
- `--no-encode` skips the encoder prompt.
- If neither flag is provided, RPG-Kit may prompt in an interactive terminal when Python code is present.

You can always run the encoder later from the AI assistant:

```text
/rpgkit.encode
```

## MCP Server

RPG-Kit's MCP server is named `rpg-tools`. It reads `.rpgkit/data/rpg.json` and exposes read-only graph-query tools to the AI assistant.

| Tool | Purpose |
| ---- | ------- |
| `search_rpg` | Search code entities or features by keyword, path, function, class, or feature name |
| `explore_rpg` | Traverse dependencies and call chains from a starting node |
| `get_node_detail` | Fetch details for a specific node, optionally including source code |
| `list_rpg_tree` | Render the functional architecture as a tree |

If `.rpgkit/data/rpg.json` does not exist yet, the tools return an `rpg_unavailable` response with a next step telling the agent to run `/rpgkit.encode`.

## Assistant Configuration Files

### Claude Code

For Claude Code, RPG-Kit writes command definitions and settings under `.claude/`:

```text
.claude/
├── commands/              # /rpgkit.* command definitions
└── settings.json          # permissions and MCP auto-approval
```

The settings file grants project-scoped permissions needed by RPG-Kit commands, including access to the `rpg-tools` MCP server. Review `.claude/settings.json` if your team wants stricter local permission prompts.

### GitHub Copilot / VS Code

For Copilot, RPG-Kit writes agent instructions under `.github/` and VS Code MCP configuration under `.vscode/`:

```text
.github/
├── agents/                # rpgkit.* agent definitions
└── prompts/               # companion prompts
.vscode/
└── mcp.json               # rpg-tools registration
```

Open the project in VS Code after initialization so the workspace MCP configuration is available to Copilot.

## Auto-approval and Scope

RPG-Kit pre-authorizes the `rpg-tools` MCP server where the selected assistant supports project-scoped permissions. The goal is to avoid prompting on every graph query during chat.

Scope rules:

- Configuration is written into the project that ran `rpgkit init` or `rpgkit update`.
- User-level assistant settings are not modified.
- Passing `--no-mcp` skips MCP registration and related auto-approval entries.

## Git Hooks and Incremental Updates

RPG-Kit installs local git hooks to keep the RPG aligned with code changes.

The important hook behavior is:

- After commits, RPG-Kit can run an incremental update in the background.
- The update refreshes `.rpgkit/data/rpg.json` and `.rpgkit/data/dep_graph.json`.
- Logs are written to `.rpgkit/logs/update_rpg.log`.

Manual fallback:

```text
/rpgkit.update_rpg
```

Use `/rpgkit.update_rpg` when:

- The hook failed.
- The hook was skipped.
- You want to force a foreground update and inspect the result.

If the RPG seems significantly stale or corrupted, run a full encode instead:

```text
/rpgkit.encode
```

## Updating an Existing RPG-Kit Project

Run `rpgkit update` from the project root to refresh scripts, command definitions, MCP configuration, gitignore rules, and hooks.

```bash
rpgkit update
rpgkit update --ai claude
rpgkit update --pre
rpgkit update --no-mcp
```

`rpgkit update` auto-detects the existing assistant configuration when possible.

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

## Troubleshooting

### AI assistant CLI not found

Run:

```bash
rpgkit check
```

Install and authenticate the missing assistant CLI, or rerun init with the assistant you want:

```bash
rpgkit init my-project --ai claude
rpgkit init my-project --ai copilot
```

### MCP tools say `rpg_unavailable`

The MCP server is configured, but `.rpgkit/data/rpg.json` has not been created yet. Run:

```text
/rpgkit.encode
```

### Incremental update failed

Check:

```bash
tail -n 200 .rpgkit/logs/update_rpg.log
```

Then run:

```text
/rpgkit.update_rpg
```

If the graph is corrupted or too stale, run `/rpgkit.encode` for a full rebuild.

### Template download hits rate limits or private repo access errors

Use a token:

```bash
rpgkit init my-project --github-token $GITHUB_TOKEN
```

or set an environment variable:

```bash
export GH_TOKEN=your_token
```
