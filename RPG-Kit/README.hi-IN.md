<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## AI-संचालित द्विदिश Repository-RPG टूलकिट

RPG-Kit LLM-आधारित एजेंटों का उपयोग करके **Repository Planning Graphs (RPG)** — एक एकीकृत ग्राफ जो फीचर्स, फाइलों, कंपोनेंट्स और dependencies को जोड़ता है — के साथ द्विदिश रूप से काम करता है:

- **Forward (Requirements → RPG → Code):** AI coding agents द्वारा संचालित multi-phase pipeline के माध्यम से high-level requirements को tested, structured repositories में बदलता है
- **Reverse (Code → RPG):** मौजूदा codebases को RPG graphs में encode करता है ताकि AI-assisted search, exploration, और incremental updates किए जा सकें
- **Surgical edits (Instruction → RPG + Code):** code, RPG, और dependency graph को synchronized रखते हुए targeted changes लागू करता है

### आगामी फीचर्स

- **सरल decoder commands:** मौजूदा decoder flow को कम commands में merge करना, जिसमें end-to-end repository generation के लिए `/rpgkit.generate_repo`, और feature generation तथा RPG planning के लिए `/rpgkit.generate_feature` plus `/rpgkit.plan` शामिल हैं।
- **Multi-language support:** Go, C++, Rust, JavaScript/TypeScript, और अन्य के लिए support जोड़ना।
- **अधिक platform integrations:** अलग-अलग systems पर अलग-अलग AI coding agents के लिए CLI और VS Code extension workflows में RPG-Kit support करना।

| प्लेटफ़ॉर्म             | Claude Code | GitHub Copilot | Codex |
| ----------------------- | ----------- | -------------- | ----- |
| CLI उपयोग               | ✅          | ✅(MCP नहीं)   | ⌛    |
| VS Code extension उपयोग | ✅          | ✅             | ⌛    |

| स्क्रिप्ट | Linux | Windows | Mac |
| --------- | ----- | ------- | --- |
| sh        | ✅    | ⌛      | ⌛  |
| ps        | N/A   | ⌛      | ⌛  |

## पूर्वापेक्षाएँ

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- installed और authenticated AI coding agent CLI: [GitHub Copilot](https://docs.github.com/en/copilot) या [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

installation के बाद local tool availability सत्यापित करने के लिए `rpgkit check` का उपयोग करें।

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

### 1. Project initialize करें

```bash
# नया project बनाएं
rpgkit init my-project

# या current directory में initialize करें
rpgkit init .

# Private repositories के लिए token आवश्यक है
rpgkit init my-project --github-token $GITHUB_TOKEN

# Existing codebases initial RPG तुरंत build कर सकते हैं
rpgkit init . --encode

# Project directory में जाएं
cd my-project
```

### 2. `/rpgkit` Commands का उपयोग करें

अपना AI coding agent launch करें और `/rpgkit.*` commands को क्रम से run करें:

```text
# Phase 1: Feature Specification
/rpgkit.feature_spec <feature description>
/rpgkit.feature_build
/rpgkit.feature_refactor
/rpgkit.feature_edit <edit instructions>       # optional, skeleton planning से पहले

# Phase 2: RPG Construction & Planning
/rpgkit.build_skeleton
/rpgkit.build_data_flow
/rpgkit.design_base_classes
/rpgkit.design_interfaces
/rpgkit.plan_tasks

# Phase 3: Code Generation
/rpgkit.code_gen
/rpgkit.rpg_edit <edit instructions>           # optional, RPG/code मौजूद होने के बाद

# Reverse Direction: existing repo को RPG में encode करें
/rpgkit.encode                                  # full encode
/rpgkit.update_rpg                              # manual incremental update fallback
```

## `/rpgkit` Commands

RPG-Kit 13 slash commands प्रदान करता है। forward pipeline requirements से code generate करता है; encoder existing code से RPG build करता है; `rpg_edit` targeted changes लागू करता है और RPG तथा code को synchronized रखता है।

> प्रत्येक command के detailed usage के लिए [docs/commands.md](docs/commands.md) देखें।

### Phase 1: Feature Specification

| Command | Description |
| ------- | ----------- |
| `/rpgkit.feature_spec <desc>` | user input या `docs/` files से structured feature specifications बनाएं |
| `/rpgkit.feature_build` | specifications से feature tree generate और expand करें |
| `/rpgkit.feature_refactor` | feature tree को modular component architecture में refactor करें |
| `/rpgkit.feature_edit <instr>` | skeleton planning से पहले feature tree nodes edit करें — optional |

### Phase 2: RPG Construction & Planning

| Command | Description |
| ------- | ----------- |
| `/rpgkit.build_skeleton` | component architecture से repository file skeleton build करें; `.rpgkit/data/rpg.json` create करता है |
| `/rpgkit.build_data_flow` | inter-component data flow DAG build करें और RPG update करें |
| `/rpgkit.design_base_classes` | shared base classes और data structures design करें |
| `/rpgkit.design_interfaces` | type hints और docstrings वाले function/class interfaces design करें |
| `/rpgkit.plan_tasks` | dependency-ordered implementation task batches plan करें |

### Phase 3: Code Generation & Surgical Edits

| Command | Description |
| ------- | ----------- |
| `/rpgkit.code_gen` | iterative test-code-fix cycles के साथ TDD-based implementation |
| `/rpgkit.rpg_edit <instr>` | natural-language instruction से RPG graph, code, और dependency graph का surgical edit — optional |

### RPG Encoder (Reverse Direction: Code → RPG)

| Command | Description |
| ------- | ----------- |
| `/rpgkit.encode` | existing repository को `.rpgkit/data/rpg.json` में encode करें |
| `/rpgkit.update_rpg` | automatic hook skip या fail होने पर manual incremental RPG update run करें |

दोनों directions `.rpgkit/data/rpg.json` पर वही RPG structure produce करते हैं, जिससे AI agents **MCP server** (`search_rpg`, `explore_rpg`, `get_node_detail`, `list_rpg_tree`) के माध्यम से graph query कर सकते हैं। नीचे [MCP Integration](#mcp-integration) देखें।

## CLI Reference

### `rpgkit init`

latest template से नया project initialize करें।

```bash
rpgkit init <project-name> [options]
rpgkit init --here [options]
rpgkit init . [options]
```

| Option | Description |
| ------ | ----------- |
| `--ai <agent>` | AI assistant: `copilot` या `claude` |
| `--script <type>` | Script type: `sh` (POSIX) या `ps` (PowerShell) |
| `--here` | current directory में initialize करें |
| `--force` | non-empty current directory के लिए confirmation skip करें |
| `--no-git` | git initialization skip करें |
| `--no-mcp` | MCP server configuration skip करें |
| `--ignore-agent-tools` | AI agent CLI tools checks skip करें |
| `--github-token <token>` | private repos या higher rate limits के लिए GitHub token |
| `--pre` | latest pre-release template download करें |
| `--skip-tls` | SSL/TLS verification skip करें |
| `--encode/--no-encode` | init के अंत में initial RPG encoding run या skip करें |
| `--debug` | verbose diagnostic output दिखाएं |

**Supported AI Assistants:**

| Agent | Folder | Description | Status |
| ----- | ------ | ----------- | ------ |
| `copilot` | `.github/`, `.vscode/` | GitHub Copilot | Verified |
| `claude` | `.claude/` | Claude Code | Verified |

> RPG-Kit currently CLI में केवल **GitHub Copilot** और **Claude Code** support करता है। Additional agents future releases में adapt किए जा सकते हैं।

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

existing project में RPG-Kit template files, scripts, command definitions, MCP configuration, gitignore rules, और hooks update करें। AI assistant संभव होने पर existing project configuration से auto-detect किया जाता है।

```bash
rpgkit update
rpgkit update --ai claude
rpgkit update --pre
rpgkit update --no-mcp
rpgkit update --github-token $GITHUB_TOKEN
```

| Option | Description |
| ------ | ----------- |
| `--ai <agent>` | AI assistant, specify न होने पर auto-detected |
| `--script <type>` | Script type: `sh` (POSIX) या `ps` (PowerShell) |
| `--github-token <token>` | private repos या higher rate limits के लिए GitHub token |
| `--pre` | latest pre-release template download करें |
| `--no-mcp` | MCP server configuration skip करें |
| `--skip-tls` | SSL/TLS verification skip करें |
| `--debug` | verbose diagnostic output दिखाएं |

### `rpgkit check`

Verify करें कि required tools installed हैं।

```bash
rpgkit check
```

### `rpgkit version`

Version और system information display करें।

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
                       │ feature_edit│ feature_tree.json में optional pre-planning edits
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
                                                  │ post-commit hook normally incremental updates चलाता है

MCP Server: search_rpg / explore_rpg / get_node_detail / list_rpg_tree
```

## Project Structure

`rpgkit init` run करने के बाद, workspace root project repository root होता है। RPG-Kit agent command definitions, runtime scripts, MCP configuration, और generated data को आपके code के साथ जोड़ता है।

```text
my-project/
├── docs/                 # /rpgkit.feature_spec के लिए optional requirement docs
├── .github/ or .claude/  # AI assistant command definitions और settings
├── .vscode/              # applicable होने पर Copilot/VS Code MCP configuration
└── .rpgkit/              # RPG-Kit runtime
    ├── scripts/          # Pipeline scripts और support packages
    ├── data/             # Generated artifacts, जिनमें rpg.json और dep_graph.json शामिल हैं
    ├── logs/             # Per-stage execution logs
    └── reports/          # Generated review और diagnostic reports
```

Full directory layout और data file reference के लिए [docs/project-structure.md](docs/project-structure.md) देखें।

## MCP Integration

`rpgkit init` automatically RPG-Kit के MCP server (`rpg-tools`) को आपके AI assistant के साथ register करता है, जब तक `--no-mcp` pass न किया जाए। Server `.rpgkit/data/rpg.json` पढ़ता है और चार read-only tools expose करता है:

| Tool | Purpose |
| ---- | ------- |
| `search_rpg` | keyword, name, path, function, class, या feature से nodes search करें |
| `explore_rpg` | starting node से dependency और call-chain edges traverse करें |
| `get_node_detail` | node का full record और optionally source code प्राप्त करें |
| `list_rpg_tree` | repository की functional architecture को tree के रूप में render करें |

MCP configuration उस project तक scoped है जिसने `rpgkit init` run किया; user-level assistant settings modify नहीं होतीं। यदि graph अभी generate नहीं हुआ है, तो MCP tools `rpg_unavailable` hint return करते हैं जो agent को `/rpgkit.encode` run करने के लिए बताता है।

MCP registration, auto-approval, hooks, और initialization options के लिए [docs/configuration.md](docs/configuration.md) देखें।

## Troubleshooting

**AI assistant CLI नहीं मिला:**

```bash
rpgkit check
```

Selected assistant CLI install और authenticate करें, फिर `rpgkit init` या `rpgkit update` दोबारा run करें।

**MCP tools `rpg_unavailable` report करते हैं:**

```text
/rpgkit.encode
```

MCP server configured है, लेकिन `.rpgkit/data/rpg.json` अभी create नहीं हुआ है।

**Incremental update failed:**

```bash
tail -n 200 .rpgkit/logs/update_rpg.log
```

फिर run करें:

```text
/rpgkit.update_rpg
```

**Rate limits या private repo access के कारण template download fail होता है:**

```bash
rpgkit init my-project --github-token $GITHUB_TOKEN
# या environment variable set करें:
export GH_TOKEN=your_token
```

## License

MIT License - विवरण के लिए [LICENSE](LICENSE) देखें।

## Acknowledgements

[GitHub Spec-Kit](https://github.com/github/spec-kit) पर आधारित।
