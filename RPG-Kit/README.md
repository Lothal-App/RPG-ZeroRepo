<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## AI-Powered Bidirectional Repository-RPG Toolkit

RPG-Kit leverages LLM-based agents to work bidirectionally with **Repository Planning Graphs (RPG)** — a unified graph connecting features, files, components, and dependencies:

- **Forward (Requirements → RPG → Code):** transforms high-level requirements into tested, structured repositories through a multi-phase pipeline powered by AI coding agents
- **Reverse (Code → RPG):** encodes existing codebases into RPG graphs for AI-assisted search, exploration, and incremental updates
- **Surgical edits (Instruction → RPG + Code):** applies targeted changes while keeping code, RPG, and dependency graph synchronized

### Upcoming Features

- **Simpler decoder commands:** merge the current decoder flow into fewer commands, including `/rpgkit.generate_repo` for end-to-end repository generation and `/rpgkit.generate_feature` plus `/rpgkit.plan` for feature generation and RPG planning.
- **Multi-language support:** add support for Go, C++, Rust, JavaScript/TypeScript, and more.
- **More platform integrations:** support RPG-Kit across CLI and VS Code extension workflows for different AI coding agents on different systems.

| Platform                | Claude Code | GitHub Copilot | Codex |
| ----------------------- | ----------- | -------------- | ----- |
| CLI usage               | ✅          | ✅(No MCP)     | ⌛    |
| VS Code extension usage | ✅          | ✅             | ⌛    |

| Script | Linux | Windows | Mac |
| ------ | ----- | ------- | --- |
| sh     | ✅    | ⌛      | ⌛  |
| ps     | N/A   | ⌛      | ⌛  |

## Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- An installed and authenticated AI coding agent CLI: [GitHub Copilot](https://docs.github.com/en/copilot) or [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

Use `rpgkit check` after installation to verify local tool availability.

## Installation

### Option 1: Persistent Installation (Recommended)

```bash
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
```

### Option 2: One-time Usage

```bash
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

## Quick Start

### 1. Initialize Project

```bash
# Create new project
rpgkit init my-project

# Or initialize in current directory
rpgkit init .

# Private repositories require token
rpgkit init my-project --github-token $GITHUB_TOKEN

# Existing codebases can build the initial RPG immediately
rpgkit init . --encode

# Enter the project directory
cd my-project
```

### 2. Use `/rpgkit` Commands

Launch your AI coding agent and run the `/rpgkit.*` commands in sequence:

```text
# Phase 1: Feature Specification
/rpgkit.feature_spec <feature description>
/rpgkit.feature_build
/rpgkit.feature_refactor
/rpgkit.feature_edit <edit instructions>       # optional, before skeleton planning

# Phase 2: RPG Construction & Planning
/rpgkit.build_skeleton
/rpgkit.build_data_flow
/rpgkit.design_base_classes
/rpgkit.design_interfaces
/rpgkit.plan_tasks

# Phase 3: Code Generation
/rpgkit.code_gen
/rpgkit.rpg_edit <edit instructions>           # optional, after RPG/code exist

# Reverse Direction: Encode existing repo → RPG
/rpgkit.encode                                  # full encode
/rpgkit.update_rpg                              # manual incremental update fallback
```

## `/rpgkit` Commands

RPG-Kit provides 13 slash commands. The forward pipeline generates code from requirements; the encoder builds an RPG from existing code; `rpg_edit` applies targeted changes while keeping RPG and code synchronized.

> For detailed usage of each command, see [docs/commands.md](docs/commands.md).

### Phase 1: Feature Specification

| Command | Description |
| ------- | ----------- |
| `/rpgkit.feature_spec <desc>` | Create structured feature specifications from user input or `docs/` files |
| `/rpgkit.feature_build` | Generate and expand the feature tree from specifications |
| `/rpgkit.feature_refactor` | Refactor feature tree into modular component architecture |
| `/rpgkit.feature_edit <instr>` | Edit feature tree nodes before skeleton planning — optional |

### Phase 2: RPG Construction & Planning

| Command | Description |
| ------- | ----------- |
| `/rpgkit.build_skeleton` | Build repository file skeleton from component architecture; creates `.rpgkit/data/rpg.json` |
| `/rpgkit.build_data_flow` | Build inter-component data flow DAG and update the RPG |
| `/rpgkit.design_base_classes` | Design shared base classes and data structures |
| `/rpgkit.design_interfaces` | Design function/class interfaces with type hints and docstrings |
| `/rpgkit.plan_tasks` | Plan dependency-ordered implementation task batches |

### Phase 3: Code Generation & Surgical Edits

| Command | Description |
| ------- | ----------- |
| `/rpgkit.code_gen` | TDD-based implementation with iterative test-code-fix cycles |
| `/rpgkit.rpg_edit <instr>` | Surgical edit of RPG graph, code, and dependency graph from a natural-language instruction — optional |

### RPG Encoder (Reverse Direction: Code → RPG)

| Command | Description |
| ------- | ----------- |
| `/rpgkit.encode` | Encode an existing repository into `.rpgkit/data/rpg.json` |
| `/rpgkit.update_rpg` | Manually run incremental RPG update when the automatic hook is skipped or fails |

Both directions produce the same RPG structure at `.rpgkit/data/rpg.json`, enabling AI agents to query the graph via the **MCP server** (`search_rpg`, `explore_rpg`, `get_node_detail`, `list_rpg_tree`). See [MCP Integration](#mcp-integration) below.

## CLI Reference

### `rpgkit init`

Initialize a new project from the latest template.

```bash
rpgkit init <project-name> [options]
rpgkit init --here [options]
rpgkit init . [options]
```

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

**Supported AI Assistants:**

| Agent | Folder | Description | Status |
| ----- | ------ | ----------- | ------ |
| `copilot` | `.github/`, `.vscode/` | GitHub Copilot | Verified |
| `claude` | `.claude/` | Claude Code | Verified |

> RPG-Kit currently supports only **GitHub Copilot** and **Claude Code** in the CLI. Additional agents may be adapted in future releases.

**Examples:**

```bash
rpgkit init my-project
rpgkit init my-project --ai claude --script sh
rpgkit init . --force
rpgkit init --here --ai copilot
rpgkit init --here --github-token $GITHUB_TOKEN
rpgkit init --here --encode
```

### `rpgkit update`

Update RPG-Kit template files, scripts, command definitions, MCP configuration, gitignore rules, and hooks in an existing project. The AI assistant is auto-detected from existing project configuration when possible.

```bash
rpgkit update
rpgkit update --ai claude
rpgkit update --pre
rpgkit update --no-mcp
rpgkit update --github-token $GITHUB_TOKEN
```

| Option | Description |
| ------ | ----------- |
| `--ai <agent>` | AI assistant, auto-detected if not specified |
| `--script <type>` | Script type: `sh` (POSIX) or `ps` (PowerShell) |
| `--github-token <token>` | GitHub token for private repos or higher rate limits |
| `--pre` | Download the latest pre-release template |
| `--no-mcp` | Skip MCP server configuration |
| `--skip-tls` | Skip SSL/TLS verification |
| `--debug` | Show verbose diagnostic output |

### `rpgkit check`

Verify that required tools are installed.

```bash
rpgkit check
```

### `rpgkit version`

Display version and system information.

```bash
rpgkit version
```

## Workflow

```text
Forward Direction: Requirements → RPG → Code

 Phase 1: Feature Specification       Phase 2: RPG Construction & Planning                             Phase 3
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ feature  │ │ feature  │ │ feature  │ │  build   │ │  build   │ │ design   │ │ design   │ │  plan    │ │          │
│  _spec   ├─▶  _build  ├─▶_refactor ├─▶ skeleton ├─▶  data    ├─▶  base    ├─▶interfaces├─▶  tasks  ├─▶ code_gen │
│          │ │          │ │          │ │          │ │  flow    │ │ classes  │ │          │ │          │ │   (TDD)  │
└──────────┘ └──────────┘ └────┬─────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └────┬─────┘
 feature_     feature_        │        skeleton     data_flow    base_        interfaces   tasks        source
 spec/        build           │        .json        .json        classes      .json        .json        code
 feature_     .json           │        skeleton_    data_flow    .json
 spec.json                    │        summary.txt  _viz.html
                              │
                       ┌──────▼──────┐
                       │ feature_edit│ optional pre-planning edits to feature_tree.json
                       └─────────────┘
                                        ╰───── rpg.json (created → progressively enriched) ─────╯
                                                                            │
                                                                            ▼
                                                                     ┌──────────┐
                                                                     │ rpg_edit │ optional synchronized RPG + code + dep_graph edits
                                                                     └──────────┘

Reverse Direction: Code → RPG

┌──────────────────┐         ┌──────────┐       ┌──────────┐
│ Existing Codebase│────────▶│  encode  │──────▶│update_rpg│
│                  │         │  (full)  │       │ (manual  │
└──────────────────┘         └──────────┘       │ fallback)│
                              rpg.json          └──────────┘
                              dep_graph.json     rpg.json / dep_graph.json
                                                  ▲
                                                  │ post-commit hook normally runs incremental updates

MCP Server: search_rpg / explore_rpg / get_node_detail / list_rpg_tree
```

## Project Structure

After running `rpgkit init`, the workspace root is the project repository root. RPG-Kit adds agent command definitions, runtime scripts, MCP configuration, and generated data alongside your code.

```text
my-project/
├── docs/                 # Optional requirement docs for /rpgkit.feature_spec
├── .github/ or .claude/  # AI assistant command definitions and settings
├── .vscode/              # Copilot/VS Code MCP configuration when applicable
└── .rpgkit/              # RPG-Kit runtime
    ├── scripts/          # Pipeline scripts and support packages
    ├── data/             # Generated artifacts, including rpg.json and dep_graph.json
    ├── logs/             # Per-stage execution logs
    └── reports/          # Review and diagnostic reports when generated
```

For the full directory layout and data file reference, see [docs/project-structure.md](docs/project-structure.md).

## MCP Integration

`rpgkit init` automatically registers RPG-Kit's MCP server (`rpg-tools`) with your AI assistant unless `--no-mcp` is passed. The server reads `.rpgkit/data/rpg.json` and exposes four read-only tools:

| Tool | Purpose |
| ---- | ------- |
| `search_rpg` | Search nodes by keyword, name, path, function, class, or feature |
| `explore_rpg` | Traverse dependency and call-chain edges from a starting node |
| `get_node_detail` | Get a node's full record and optionally source code |
| `list_rpg_tree` | Render the repository's functional architecture as a tree |

The MCP configuration is scoped to the project that ran `rpgkit init`; user-level assistant settings are not modified. If the graph has not been generated yet, MCP tools return an `rpg_unavailable` hint telling the agent to run `/rpgkit.encode`.

For MCP registration, auto-approval, hooks, and initialization options, see [docs/configuration.md](docs/configuration.md).

## Troubleshooting

**AI assistant CLI not found:**

```bash
rpgkit check
```

Install and authenticate the selected assistant CLI, then rerun `rpgkit init` or `rpgkit update`.

**MCP tools report `rpg_unavailable`:**

```text
/rpgkit.encode
```

The MCP server is configured, but `.rpgkit/data/rpg.json` has not been created yet.

**Incremental update failed:**

```bash
tail -n 200 .rpgkit/logs/update_rpg.log
```

Then run:

```text
/rpgkit.update_rpg
```

**Template download fails due to rate limits or private repo access:**

```bash
rpgkit init my-project --github-token $GITHUB_TOKEN
# or set the environment variable:
export GH_TOKEN=your_token
```

## License

MIT License - See [LICENSE](LICENSE) for details.

## Acknowledgements

Based on [GitHub Spec-Kit](https://github.com/github/spec-kit).
