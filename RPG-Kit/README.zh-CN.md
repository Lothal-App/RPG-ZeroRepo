<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## 让编码智能体先规划，再编辑

编码智能体擅长局部编辑，但仓库级任务如果缺少稳定的规划结构往往会失败：需求漂移、架构决策丢失、多文件生成前后不一致、更新可能错过隐藏依赖。

RPG-Kit 为 Claude Code 和 GitHub Copilot 提供一个面向仓库级编码的**持久化 RPG 工作区**。这个工作区围绕一个 **Repository Planning Graph (RPG)** 构建，把需求、功能、架构、文件、代码实体和依赖关系连接在一起。

借助 RPG-Kit，智能体可以通过图驱动的工作流来工作：

- **构建（Build）**：把需求转换为 RPG 规划，然后生成一个多文件仓库。
- **理解（Understand）**：把已有仓库映射为 RPG，然后搜索、浏览和解释它。
- **更新（Update）**：定位受影响的 RPG 节点，规划编辑，并同步更新代码和图。

### 选择你的工作流

| 目标 | 工作流 | 从这里开始 |
|---|---|---|
| 从需求构建一个新仓库 | Build 工作流（requirements → RPG → code） | [`快速开始：新仓库`](#快速开始新仓库) |
| 理解一个已有仓库 | Understand 工作流（repository → RPG → search/explore） | [`快速开始：已有仓库`](#快速开始已有仓库) |
| 更新一个已有仓库 | Update 工作流（change request → affected RPG nodes → edit plan → code/RPG update） | [`快速开始：已有仓库`](#快速开始已有仓库) |

### 详细流水线

新用户可以跳过这一节，直接从下面的「快速开始」开始。

<details>
<summary>完整的命令级工作流图</summary>

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
Surgical edit workflow: Requirements -> RPG update -> Code Update    │ rpg_edit │ optional synchronized RPG + code + dep_graph edits
                                                                     └──▲────▲──┘
                                                                        │    │
Reverse Direction: Code → RPG                                           │    │
                                                                        │    │
┌──────────────────┐         ┌──────────┐       ┌──────────┐            │    │
│ Existing Codebase│────────▶│  encode  │──────▶│update_rpg│────────────┘    │
│                  │         │  (full)  │       │ (manual  │                 │
└──────────────────┘         └────┬─────┘       │ fallback)│                 │
                              rpg.json          └──────────┘                 │
                              dep_graph.json     rpg.json / dep_graph.json   │
                                  │                                          │
                                  └──────────────────────────────────────────┘
                                                  ▲
                                                  │ post-commit hook normally runs incremental updates

MCP Server: search_rpg / explore_rpg / get_node_detail / list_rpg_tree
```

</details>

### RPG-Kit 实际效果

下图是为本仓库生成的图可视化的一部分。运行 `/rpgkit.encode`，然后打开 `.rpgkit/data/rpg.html` 浏览完整的交互式图。

![RPG-Kit repository graph visualization](../docs/rpgkit_visualized_graph.png)

## 安装

### 先决条件

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- 一个已安装并完成身份验证的 AI 编码智能体 CLI：[GitHub Copilot](https://docs.github.com/en/copilot) 或 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

### 安装 RPG-Kit

```bash
# 持久化安装（推荐）
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
rpgkit check

# 一次性使用
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

## 快速开始：新仓库

当你希望 RPG-Kit 把需求转换为新代码库时，使用此路径。

> [!WARNING]
> 对于生成代码量较大的项目，`/rpgkit.design_interfaces` 和 `/rpgkit.code_gen` 可能运行较长时间。典型例子：100 个 feature 大约需要 30 分钟。

1. 初始化一个新项目：

   ```bash
   rpgkit init my-project
   cd my-project
   ```

   常见变体：

   ```bash
   rpgkit init my-project --ai claude --script sh
   rpgkit init my-project --ai copilot
   rpgkit init my-project --github-token $GITHUB_TOKEN
   ```

2. **[可选]** 把你的需求文档放在 `my-project/docs/`。

3. 在项目目录里启动你的 AI 编码智能体。

4. 运行正向流水线：

   ```text
   /rpgkit.feature_spec <feature description>
   /rpgkit.feature_build
   /rpgkit.feature_refactor
   [Optional] /rpgkit.feature_edit <edit instructions>
   /rpgkit.build_skeleton
   /rpgkit.build_data_flow
   /rpgkit.design_base_classes
   /rpgkit.design_interfaces
   /rpgkit.plan_tasks
   /rpgkit.code_gen
   [Optional] /rpgkit.rpg_edit <edit instructions>
   ```

RPG-Kit 会渐进式地创建 `.rpgkit/data/rpg.json`，并用它把需求、规划产物、生成的代码和依赖信息保持对齐。

## 快速开始：已有仓库

当你已经有一个仓库，希望 AI 智能体在 RPG 上下文中理解或编辑它时，使用此路径。

> [!WARNING]
> 对于较大的项目，`rpgkit init . --encode` 和 `/rpgkit.encode` 可能运行较长时间。典型例子：200 个源文件大约需要 100 分钟。

1. 在仓库根目录初始化 RPG-Kit 并构建初始图：

   ```bash
   mkdir my-project
   cp -r existing-repo/ my-project/
   cd my-project
   rpgkit init . --encode
   ```

   如果你想跳过非空目录的确认提示：

   ```bash
   rpgkit init . --force --encode
   ```

2. 在仓库里启动你的 AI 编码智能体。

3. 通过 MCP 工具和 slash 命令使用生成的 RPG：

   ```text
   /rpgkit.encode                                  # 需要时重建完整 RPG
   /rpgkit.update_rpg                              # 手动增量更新（fallback）
   /rpgkit.rpg_edit <edit instructions>            # 图感知的代码编辑
   ```

4. 提交后，RPG-Kit hook 会把 `.rpgkit/data/rpg.json`、`.rpgkit/data/dep_graph.json` 和 `.rpgkit/data/rpg.html` 与代码变更保持对齐。如果 hook 失败或被跳过，运行 `/rpgkit.update_rpg`。

## `rpgkit init` 之后会发生什么

`rpgkit init` 不会修改你的源文件。它会在你的代码旁边添加命令定义、运行时脚本、MCP 配置和生成的图数据。

```text
my-project/
├── docs/                 # /rpgkit.feature_spec 的可选需求文档
├── .github/ or .claude/  # AI 助手的命令定义和设置
├── .vscode/              # 适用时的 Copilot/VS Code MCP 配置
└── .rpgkit/              # RPG-Kit 运行时
    ├── scripts/          # 流水线脚本和支持包
    ├── data/             # 生成的产物，包括 rpg.json 和 dep_graph.json
    ├── logs/             # 各阶段执行日志
    └── reports/          # 生成时的审查与诊断报告
```

完整的目录布局和数据文件参考见 [docs/project-structure.md](docs/project-structure.md)。

## 支持的平台

| 平台                | Claude Code | GitHub Copilot | Codex |
| ------------------- | ----------- | -------------- | ----- |
| CLI 使用            | ✅          | ✅ (No MCP)    | ⌛    |
| VS Code 扩展使用    | ✅          | ✅             | ⌛    |

| 脚本 | Linux | Windows | Mac |
| ---- | ----- | ------- | --- |
| sh   | ✅    | ⌛      | ⌛  |
| ps   | N/A   | ⌛      | ⌛  |

## 文档

- [Slash 命令参考](docs/commands.md) —— 每一个 `/rpgkit.*` 命令的输入、输出和示例。
- [CLI 参考](docs/cli-reference.md) —— `rpgkit init`、`rpgkit update`、`rpgkit check`、`rpgkit version` 以及所有选项。
- [配置](docs/configuration.md) —— AI 助手设置、MCP 注册、hook、自动审批和故障排查。
- [项目结构](docs/project-structure.md) —— RPG-Kit 创建的文件和目录。

## 即将推出的功能

- **更简化的生成命令**：把当前多步骤的生成流程合并为更少的命令，例如 `/rpgkit.generate_repo`、`/rpgkit.generate_feature` 和 `/rpgkit.plan`。
- **多语言支持**：增加对 Go、C++、Rust、JavaScript/TypeScript 等的支持。
- **更多平台集成**：在不同系统上跨 CLI 和 VS Code 扩展工作流支持不同的 AI 编码智能体。

## 故障排查

**找不到 AI 助手 CLI**：运行 `rpgkit check`，安装并完成所选助手 CLI 的身份验证，然后重新运行 `rpgkit init` 或 `rpgkit update`。

**MCP 工具报告 `rpg_unavailable`**：运行 `/rpgkit.encode` 来创建 `.rpgkit/data/rpg.json`。

**增量更新失败**：检查 `.rpgkit/logs/update_rpg.log`，然后运行 `/rpgkit.update_rpg`。

**因为速率限制或私有仓库访问导致模板下载失败**：传递 `--github-token $GITHUB_TOKEN`，或设置 `GH_TOKEN` / `GITHUB_TOKEN`。

## 许可证

MIT License —— 详情见 [LICENSE](LICENSE)。

## 致谢

基于 [GitHub Spec-Kit](https://github.com/github/spec-kit)。
