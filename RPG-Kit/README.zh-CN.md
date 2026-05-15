<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## AI 驱动的双向 Repository-RPG 工具包

RPG-Kit 利用基于 LLM 的智能体，以双向方式处理 **Repository Planning Graphs (RPG)** —— 一种连接功能、文件、组件和依赖关系的统一图结构：

- **正向（需求 → RPG → 代码）：** 通过由 AI 编码智能体驱动的多阶段流水线，将高层需求转换为经过测试、结构化的代码仓库
- **反向（代码 → RPG）：** 将现有代码库编码为 RPG 图，用于 AI 辅助搜索、探索和增量更新
- **外科式编辑（指令 → RPG + 代码）：** 应用有针对性的修改，同时保持代码、RPG 和依赖图同步

### 即将推出的功能

- **更简单的解码器命令：** 将当前解码器流程合并为更少的命令，包括用于端到端仓库生成的 `/rpgkit.generate_repo`，以及用于功能生成和 RPG 规划的 `/rpgkit.generate_feature` 加 `/rpgkit.plan`。
- **多语言支持：** 增加对 Go、C++、Rust、JavaScript/TypeScript 等语言的支持。
- **更多平台集成：** 在不同系统上，为不同 AI 编码智能体支持 RPG-Kit 的 CLI 和 VS Code 扩展工作流。

| 平台                    | Claude Code | GitHub Copilot | Codex |
| ----------------------- | ----------- | -------------- | ----- |
| CLI 使用                | ✅          | ✅(无 MCP)     | ⌛    |
| VS Code 扩展使用        | ✅          | ✅             | ⌛    |

| 脚本 | Linux | Windows | Mac |
| ---- | ----- | ------- | --- |
| sh   | ✅    | ⌛      | ⌛  |
| ps   | N/A   | ⌛      | ⌛  |

## 运行环境

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- 已安装并完成身份验证的 AI 编码智能体 CLI：[GitHub Copilot](https://docs.github.com/en/copilot) 或 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

安装后使用 `rpgkit check` 验证本地工具可用性。

## 安装

### 选项 1：持久安装（推荐）

```bash
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
```

### 选项 2：一次性使用

```bash
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

## 快速开始

### 1. 初始化项目

```bash
# 创建新项目
rpgkit init my-project

# 或在当前目录中初始化
rpgkit init .

# 私有仓库需要 token
rpgkit init my-project --github-token $GITHUB_TOKEN

# 现有代码库可以立即构建初始 RPG
rpgkit init . --encode

# 进入项目目录
cd my-project
```

### 2. 使用 `/rpgkit` 命令

启动你的 AI 编码智能体，并按顺序运行 `/rpgkit.*` 命令：

```text
# 阶段 1：功能规格
/rpgkit.feature_spec <feature description>
/rpgkit.feature_build
/rpgkit.feature_refactor
/rpgkit.feature_edit <edit instructions>       # 可选，在骨架规划之前

# 阶段 2：RPG 构建与规划
/rpgkit.build_skeleton
/rpgkit.build_data_flow
/rpgkit.design_base_classes
/rpgkit.design_interfaces
/rpgkit.plan_tasks

# 阶段 3：代码生成
/rpgkit.code_gen
/rpgkit.rpg_edit <edit instructions>           # 可选，在 RPG/代码已存在之后

# 反向：将现有仓库编码为 RPG
/rpgkit.encode                                  # 完整编码
/rpgkit.update_rpg                              # 手动增量更新兜底
```

## `/rpgkit` 命令

RPG-Kit 提供 13 个斜杠命令。正向流水线根据需求生成代码；编码器根据现有代码构建 RPG；`rpg_edit` 应用有针对性的修改，同时保持 RPG 和代码同步。

> 每个命令的详细用法见 [docs/commands.md](docs/commands.md)。

### 阶段 1：功能规格

| 命令 | 描述 |
| ---- | ---- |
| `/rpgkit.feature_spec <desc>` | 根据用户输入或 `docs/` 文件创建结构化功能规格 |
| `/rpgkit.feature_build` | 根据规格生成并扩展功能树 |
| `/rpgkit.feature_refactor` | 将功能树重构为模块化组件架构 |
| `/rpgkit.feature_edit <instr>` | 在骨架规划之前编辑功能树节点 — 可选 |

### 阶段 2：RPG 构建与规划

| 命令 | 描述 |
| ---- | ---- |
| `/rpgkit.build_skeleton` | 根据组件架构构建仓库文件骨架；创建 `.rpgkit/data/rpg.json` |
| `/rpgkit.build_data_flow` | 构建组件间数据流 DAG 并更新 RPG |
| `/rpgkit.design_base_classes` | 设计共享基类和数据结构 |
| `/rpgkit.design_interfaces` | 设计带类型提示和文档字符串的函数/类接口 |
| `/rpgkit.plan_tasks` | 规划按依赖顺序排列的实现任务批次 |

### 阶段 3：代码生成与外科式编辑

| 命令 | 描述 |
| ---- | ---- |
| `/rpgkit.code_gen` | 基于 TDD 的实现，包含迭代式测试-代码-修复循环 |
| `/rpgkit.rpg_edit <instr>` | 根据自然语言指令对 RPG 图、代码和依赖图进行外科式编辑 — 可选 |

### RPG 编码器（反向：代码 → RPG）

| 命令 | 描述 |
| ---- | ---- |
| `/rpgkit.encode` | 将现有仓库编码为 `.rpgkit/data/rpg.json` |
| `/rpgkit.update_rpg` | 当自动 hook 被跳过或失败时，手动运行增量 RPG 更新 |

两个方向都会在 `.rpgkit/data/rpg.json` 生成相同的 RPG 结构，使 AI 智能体能够通过 **MCP server**（`search_rpg`、`explore_rpg`、`get_node_detail`、`list_rpg_tree`）查询图。见下方 [MCP 集成](#mcp-集成)。

## CLI 参考

### `rpgkit init`

从最新模板初始化一个新项目。

```bash
rpgkit init <project-name> [options]
rpgkit init --here [options]
rpgkit init . [options]
```

| 选项 | 描述 |
| ---- | ---- |
| `--ai <agent>` | AI assistant：`copilot` 或 `claude` |
| `--script <type>` | 脚本类型：`sh`（POSIX）或 `ps`（PowerShell） |
| `--here` | 在当前目录初始化 |
| `--force` | 对非空当前目录跳过确认 |
| `--no-git` | 跳过 git 初始化 |
| `--no-mcp` | 跳过 MCP server 配置 |
| `--ignore-agent-tools` | 跳过 AI 智能体 CLI 工具检查 |
| `--github-token <token>` | 用于私有仓库或更高速率限制的 GitHub token |
| `--pre` | 下载最新预发布模板 |
| `--skip-tls` | 跳过 SSL/TLS 验证 |
| `--encode/--no-encode` | 在 init 结束时运行或跳过初始 RPG 编码 |
| `--debug` | 显示详细诊断输出 |

**支持的 AI Assistants：**

| 智能体 | 文件夹 | 描述 | 状态 |
| ------ | ------ | ---- | ---- |
| `copilot` | `.github/`, `.vscode/` | GitHub Copilot | 已验证 |
| `claude` | `.claude/` | Claude Code | 已验证 |

> RPG-Kit 目前在 CLI 中仅支持 **GitHub Copilot** 和 **Claude Code**。未来版本可能会适配其他智能体。

**示例：**

```bash
rpgkit init my-project
rpgkit init my-project --ai claude --script sh
rpgkit init . --force
rpgkit init --here --ai copilot
rpgkit init --here --github-token $GITHUB_TOKEN
rpgkit init --here --encode
```

### `rpgkit update`

更新现有项目中的 RPG-Kit 模板文件、脚本、命令定义、MCP 配置、gitignore 规则和 hooks。AI assistant 会尽可能从现有项目配置中自动检测。

```bash
rpgkit update
rpgkit update --ai claude
rpgkit update --pre
rpgkit update --no-mcp
rpgkit update --github-token $GITHUB_TOKEN
```

| 选项 | 描述 |
| ---- | ---- |
| `--ai <agent>` | AI assistant，如未指定则自动检测 |
| `--script <type>` | 脚本类型：`sh`（POSIX）或 `ps`（PowerShell） |
| `--github-token <token>` | 用于私有仓库或更高速率限制的 GitHub token |
| `--pre` | 下载最新预发布模板 |
| `--no-mcp` | 跳过 MCP server 配置 |
| `--skip-tls` | 跳过 SSL/TLS 验证 |
| `--debug` | 显示详细诊断输出 |

### `rpgkit check`

验证所需工具是否已安装。

```bash
rpgkit check
```

### `rpgkit version`

显示版本和系统信息。

```bash
rpgkit version
```

## 工作流

```text
正向：需求 → RPG → 代码

 阶段 1：功能规格                 阶段 2：RPG 构建与规划                                      阶段 3
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
                       │ feature_edit│ 功能树 feature_tree.json 的可选预规划编辑
                       └─────────────┘
                                        ╰───── rpg.json（创建 → 逐步丰富） ─────╯
                                                                            │
                                                                            ▼
                                                                     ┌──────────┐
                                                                     │ rpg_edit │ 可选的同步 RPG + 代码 + dep_graph 编辑
                                                                     └──────────┘

反向：代码 → RPG

┌──────────────────┐         ┌──────────┐       ┌──────────┐
│ Existing Codebase│────────▶│  encode  │──────▶│update_rpg│
│                  │         │  (full)  │       │ (manual  │
└──────────────────┘         └──────────┘       │ fallback)│
                              rpg.json          └──────────┘
                              dep_graph.json     rpg.json / dep_graph.json
                                                  ▲
                                                  │ post-commit hook 通常会运行增量更新

MCP Server: search_rpg / explore_rpg / get_node_detail / list_rpg_tree
```

## 项目结构

运行 `rpgkit init` 后，workspace root 就是项目仓库根目录。RPG-Kit 会把智能体命令定义、运行时脚本、MCP 配置和生成数据与你的代码一起添加到项目中。

```text
my-project/
├── docs/                 # /rpgkit.feature_spec 的可选需求文档
├── .github/ or .claude/  # AI assistant 命令定义和设置
├── .vscode/              # 适用时的 Copilot/VS Code MCP 配置
└── .rpgkit/              # RPG-Kit 运行时
    ├── scripts/          # 流水线脚本和支持包
    ├── data/             # 生成产物，包括 rpg.json 和 dep_graph.json
    ├── logs/             # 每个阶段的执行日志
    └── reports/          # 生成的评审和诊断报告
```

完整目录布局和数据文件参考见 [docs/project-structure.md](docs/project-structure.md)。

## MCP 集成

`rpgkit init` 会自动将 RPG-Kit 的 MCP server（`rpg-tools`）注册到你的 AI assistant，除非传入 `--no-mcp`。该 server 读取 `.rpgkit/data/rpg.json`，并暴露四个只读工具：

| 工具 | 用途 |
| ---- | ---- |
| `search_rpg` | 按关键字、名称、路径、函数、类或功能搜索节点 |
| `explore_rpg` | 从起始节点遍历依赖边和调用链边 |
| `get_node_detail` | 获取节点的完整记录，并可选择获取源代码 |
| `list_rpg_tree` | 以树形式渲染仓库的功能架构 |

MCP 配置限定在运行 `rpgkit init` 的项目范围内；不会修改用户级 assistant 设置。如果图尚未生成，MCP 工具会返回 `rpg_unavailable` 提示，告知智能体运行 `/rpgkit.encode`。

关于 MCP 注册、自动批准、hooks 和初始化选项，见 [docs/configuration.md](docs/configuration.md)。

## 故障排除

**找不到 AI assistant CLI：**

```bash
rpgkit check
```

安装并认证所选 assistant CLI，然后重新运行 `rpgkit init` 或 `rpgkit update`。

**MCP 工具报告 `rpg_unavailable`：**

```text
/rpgkit.encode
```

MCP server 已配置，但尚未创建 `.rpgkit/data/rpg.json`。

**增量更新失败：**

```bash
tail -n 200 .rpgkit/logs/update_rpg.log
```

然后运行：

```text
/rpgkit.update_rpg
```

**由于速率限制或私有仓库访问导致模板下载失败：**

```bash
rpgkit init my-project --github-token $GITHUB_TOKEN
# 或设置环境变量：
export GH_TOKEN=your_token
```

## 许可证

MIT License - 详见 [LICENSE](LICENSE)。

## 致谢

基于 [GitHub Spec-Kit](https://github.com/github/spec-kit)。
