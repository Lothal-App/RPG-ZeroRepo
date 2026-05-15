<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## 让 AI 编码智能体理解整个仓库

AI 编码智能体很强大，但它们通常逐文件工作。随着项目增长，它们可能会丢失对需求、架构、依赖关系和既有设计决策的把握。

RPG-Kit 通过维护一个 **Repository Planning Graph (RPG)** 来帮助解决这个问题：这是一张结构化地图，连接需求、功能、文件、组件和依赖关系。

当你希望 AI 智能体基于仓库级上下文工作，而不是依赖孤立的提示时，可以使用 RPG-Kit。

### 为什么选择 RPG-Kit？

| AI 编码智能体的常见问题 | RPG-Kit 如何帮助 |
|---|---|
| 智能体在几轮提示后忘记需求 | 需求会被编码进 RPG |
| 智能体在不了解相关文件的情况下编辑单个文件 | 文件、组件和依赖关系会在图中连接起来 |
| 生成的代码逐渐偏离原始计划 | 规划产物和代码会保持一致 |
| 现有仓库很难让智能体理解 | 可以将代码库编码为 RPG |
| 有针对性的编辑可能破坏隐藏依赖 | 编辑会基于图感知上下文进行 |

### 选择你的工作流

| 目标 | 工作流 | 从这里开始 |
|---|---|---|
| 从需求创建新项目 | 正向工作流 | [`快速开始：新仓库`](#quick-start-new-repository) |
| 理解或更新现有代码库 | 反向工作流 | [`快速开始：现有仓库`](#quick-start-existing-repository) |
| 进行精确的仓库感知编辑 | 外科式编辑工作流 | [`快速开始：现有仓库`](#quick-start-existing-repository) |

## 安装

### 先决条件

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- 已安装并完成身份验证的 AI 编码智能体 CLI：[GitHub Copilot](https://docs.github.com/en/copilot) 或 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

### 安装 RPG-Kit

```bash
# 持久安装（推荐）
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
rpgkit check

# 一次性使用
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

<a id="quick-start-new-repository"></a>
## 快速开始：新仓库

当你想让 RPG-Kit 将需求转换为新代码库时，使用此路径。

> [!WARNING]
> 对于生成代码量比较大的项目，`/rpgkit.design_interfaces` 和 `/rpgkit.code_gen` 的运行时间会比较长。一个典型的例子：特征数为100，运行时间大约30分钟。

1. 初始化新项目：

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

2. **[可选]** 将你的需求文档放入 `my-project/docs/`。

3. 在项目目录中启动你的 AI 编码智能体。

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

RPG-Kit 会逐步创建 `.rpgkit/data/rpg.json`，并使用它来保持需求、规划产物、生成的代码和依赖信息一致。

<a id="quick-start-existing-repository"></a>
## 快速开始：现有仓库

当你已经有一个代码仓库，并希望 AI 智能体借助 RPG 上下文理解或编辑它时，使用此路径。

> [!WARNING]
> 对于比较大的项目，`rpgkit init . --encode` 和 `/rpgkit.encode` 的运行时间可能会比较长。一个典型的例子：源代码文件数为200，运行时间100分钟。

1. 在仓库根目录初始化 RPG-Kit，并构建初始图：

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

2. 在仓库中启动你的 AI 编码智能体。

3. 通过 MCP 工具和斜杠命令使用生成的 RPG：

   ```text
   /rpgkit.encode                                  # 需要时重建完整 RPG
   /rpgkit.update_rpg                              # 手动增量更新兜底
   /rpgkit.rpg_edit <edit instructions>            # 图感知代码编辑
   ```

4. 提交后，RPG-Kit hooks 会保持 `.rpgkit/data/rpg.json`、`.rpgkit/data/dep_graph.json` 和 `.rpgkit/data/rpg.html` 与代码改动一致。如果 hook 失败或被跳过，请运行 `/rpgkit.update_rpg`。

## 新增内容

运行 `rpgkit init` 后，workspace root 仍然是你的项目仓库根目录。RPG-Kit 会将命令定义、运行时脚本、MCP 配置和生成的图数据与代码一起添加到项目中。

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

## 支持的平台

| 平台                    | Claude Code | GitHub Copilot | Codex |
| ----------------------- | ----------- | -------------- | ----- |
| CLI 使用                | ✅          | ✅(无 MCP)     | ⌛    |
| VS Code 扩展使用        | ✅          | ✅             | ⌛    |

| 脚本 | Linux | Windows | Mac |
| ---- | ----- | ------- | --- |
| sh   | ✅    | ⌛      | ⌛  |
| ps   | N/A   | ⌛      | ⌛  |

## 文档

- [斜杠命令参考](docs/commands.md) — 每个 `/rpgkit.*` 命令、输入、输出和示例。
- [CLI 参考](docs/cli-reference.md) — `rpgkit init`、`rpgkit update`、`rpgkit check`、`rpgkit version` 以及所有选项。
- [配置](docs/configuration.md) — AI assistant 设置、MCP 注册、hooks、自动批准和故障排除。
- [项目结构](docs/project-structure.md) — RPG-Kit 创建的文件和目录。

## 即将推出的功能

- **更简单的解码器命令：** 将当前解码器流程合并为更少的命令，包括用于端到端仓库生成的 `/rpgkit.generate_repo`，以及用于功能生成和 RPG 规划的 `/rpgkit.generate_feature` 加 `/rpgkit.plan`。
- **多语言支持：** 增加对 Go、C++、Rust、JavaScript/TypeScript 等语言的支持。
- **更多平台集成：** 支持 RPG-Kit 在不同系统上与不同 AI 编码智能体的 CLI 和 VS Code 扩展工作流配合使用。

## 故障排除

**找不到 AI assistant CLI：** 运行 `rpgkit check`，安装并认证所选 assistant CLI，然后重新运行 `rpgkit init` 或 `rpgkit update`。

**MCP 工具报告 `rpg_unavailable`：** 运行 `/rpgkit.encode` 来创建 `.rpgkit/data/rpg.json`。

**增量更新失败：** 检查 `.rpgkit/logs/update_rpg.log`，然后运行 `/rpgkit.update_rpg`。

**由于速率限制或私有仓库访问导致模板下载失败：** 传入 `--github-token $GITHUB_TOKEN`，或设置 `GH_TOKEN` / `GITHUB_TOKEN`。

## 许可证

MIT License - 详见 [LICENSE](LICENSE)。

## 致谢

基于 [GitHub Spec-Kit](https://github.com/github/spec-kit)。
