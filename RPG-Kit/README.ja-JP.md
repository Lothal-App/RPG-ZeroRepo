<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## AI 駆動の双方向 Repository-RPG ツールキット

RPG-Kit は LLM ベースのエージェントを活用し、**Repository Planning Graphs (RPG)** —— 機能、ファイル、コンポーネント、依存関係を接続する統一グラフ —— を双方向に扱います：

- **順方向（要件 → RPG → コード）：** AI コーディングエージェントによる多段階パイプラインを通じて、高レベルの要件をテスト済みで構造化されたリポジトリに変換します
- **逆方向（コード → RPG）：** 既存のコードベースを RPG グラフにエンコードし、AI 支援の検索、探索、インクリメンタル更新に利用します
- **外科的編集（指示 → RPG + コード）：** コード、RPG、依存グラフの同期を保ちながら、対象を絞った変更を適用します

### 今後の機能

- **よりシンプルなデコーダーコマンド：** 現在のデコーダーフローをより少ないコマンドに統合します。これには、エンドツーエンドのリポジトリ生成用の `/rpgkit.generate_repo`、および機能生成と RPG 計画用の `/rpgkit.generate_feature` と `/rpgkit.plan` が含まれます。
- **多言語サポート：** Go、C++、Rust、JavaScript/TypeScript などのサポートを追加します。
- **より多くのプラットフォーム統合：** さまざまなシステム上で、異なる AI コーディングエージェント向けに CLI と VS Code 拡張ワークフローで RPG-Kit をサポートします。

| プラットフォーム        | Claude Code | GitHub Copilot | Codex |
| ----------------------- | ----------- | -------------- | ----- |
| CLI 使用                | ✅          | ✅(MCP なし)   | ⌛    |
| VS Code 拡張の使用      | ✅          | ✅             | ⌛    |

| スクリプト | Linux | Windows | Mac |
| ---------- | ----- | ------- | --- |
| sh         | ✅    | ⌛      | ⌛  |
| ps         | N/A   | ⌛      | ⌛  |

## 前提条件

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- インストール済みで認証済みの AI コーディングエージェント CLI：[GitHub Copilot](https://docs.github.com/en/copilot) または [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

インストール後、`rpgkit check` を使用してローカルツールが利用可能か確認します。

## インストール

### オプション 1：永続インストール（推奨）

```bash
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
```

### オプション 2：一回限りの使用

```bash
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

## クイックスタート

### 1. プロジェクトを初期化する

```bash
# 新しいプロジェクトを作成
rpgkit init my-project

# または現在のディレクトリで初期化
rpgkit init .

# プライベートリポジトリには token が必要
rpgkit init my-project --github-token $GITHUB_TOKEN

# 既存のコードベースでは初期 RPG をすぐに構築可能
rpgkit init . --encode

# プロジェクトディレクトリに移動
cd my-project
```

### 2. `/rpgkit` コマンドを使用する

AI コーディングエージェントを起動し、`/rpgkit.*` コマンドを順番に実行します：

```text
# フェーズ 1：機能仕様
/rpgkit.feature_spec <feature description>
/rpgkit.feature_build
/rpgkit.feature_refactor
/rpgkit.feature_edit <edit instructions>       # 任意、スケルトン計画の前

# フェーズ 2：RPG 構築と計画
/rpgkit.build_skeleton
/rpgkit.build_data_flow
/rpgkit.design_base_classes
/rpgkit.design_interfaces
/rpgkit.plan_tasks

# フェーズ 3：コード生成
/rpgkit.code_gen
/rpgkit.rpg_edit <edit instructions>           # 任意、RPG/コードが存在した後

# 逆方向：既存リポジトリ → RPG にエンコード
/rpgkit.encode                                  # フルエンコード
/rpgkit.update_rpg                              # 手動インクリメンタル更新のフォールバック
```

## `/rpgkit` コマンド

RPG-Kit は 13 個のスラッシュコマンドを提供します。順方向パイプラインは要件からコードを生成します。エンコーダーは既存コードから RPG を構築します。`rpg_edit` は対象を絞った変更を適用しながら、RPG とコードの同期を保ちます。

> 各コマンドの詳細な使い方は [docs/commands.md](docs/commands.md) を参照してください。

### フェーズ 1：機能仕様

| コマンド | 説明 |
| -------- | ---- |
| `/rpgkit.feature_spec <desc>` | ユーザー入力または `docs/` ファイルから構造化された機能仕様を作成します |
| `/rpgkit.feature_build` | 仕様から機能ツリーを生成して拡張します |
| `/rpgkit.feature_refactor` | 機能ツリーをモジュール化されたコンポーネントアーキテクチャにリファクタリングします |
| `/rpgkit.feature_edit <instr>` | スケルトン計画の前に機能ツリーノードを編集します — 任意 |

### フェーズ 2：RPG 構築と計画

| コマンド | 説明 |
| -------- | ---- |
| `/rpgkit.build_skeleton` | コンポーネントアーキテクチャからリポジトリのファイルスケルトンを構築します；`.rpgkit/data/rpg.json` を作成します |
| `/rpgkit.build_data_flow` | コンポーネント間データフロー DAG を構築し、RPG を更新します |
| `/rpgkit.design_base_classes` | 共有基底クラスとデータ構造を設計します |
| `/rpgkit.design_interfaces` | 型ヒントと docstring を含む関数/クラスインターフェースを設計します |
| `/rpgkit.plan_tasks` | 依存関係順の実装タスクバッチを計画します |

### フェーズ 3：コード生成と外科的編集

| コマンド | 説明 |
| -------- | ---- |
| `/rpgkit.code_gen` | 反復的な test-code-fix サイクルを含む TDD ベースの実装 |
| `/rpgkit.rpg_edit <instr>` | 自然言語の指示から RPG グラフ、コード、依存グラフを外科的に編集します — 任意 |

### RPG エンコーダー（逆方向：コード → RPG）

| コマンド | 説明 |
| -------- | ---- |
| `/rpgkit.encode` | 既存リポジトリを `.rpgkit/data/rpg.json` にエンコードします |
| `/rpgkit.update_rpg` | 自動 hook がスキップまたは失敗したときに、インクリメンタル RPG 更新を手動で実行します |

両方向とも `.rpgkit/data/rpg.json` に同じ RPG 構造を生成し、AI エージェントが **MCP server**（`search_rpg`、`explore_rpg`、`get_node_detail`、`list_rpg_tree`）を通じてグラフをクエリできるようにします。下の [MCP 連携](#mcp-連携) を参照してください。

## CLI リファレンス

### `rpgkit init`

最新テンプレートから新しいプロジェクトを初期化します。

```bash
rpgkit init <project-name> [options]
rpgkit init --here [options]
rpgkit init . [options]
```

| オプション | 説明 |
| ---------- | ---- |
| `--ai <agent>` | AI assistant：`copilot` または `claude` |
| `--script <type>` | スクリプトタイプ：`sh`（POSIX）または `ps`（PowerShell） |
| `--here` | 現在のディレクトリで初期化します |
| `--force` | 空でない現在のディレクトリに対する確認をスキップします |
| `--no-git` | git 初期化をスキップします |
| `--no-mcp` | MCP server 設定をスキップします |
| `--ignore-agent-tools` | AI エージェント CLI ツールのチェックをスキップします |
| `--github-token <token>` | プライベートリポジトリまたはより高いレート制限用の GitHub token |
| `--pre` | 最新のプレリリーステンプレートをダウンロードします |
| `--skip-tls` | SSL/TLS 検証をスキップします |
| `--encode/--no-encode` | init の最後に初期 RPG エンコードを実行またはスキップします |
| `--debug` | 詳細な診断出力を表示します |

**サポートされる AI Assistants：**

| エージェント | フォルダー | 説明 | 状態 |
| ------------ | ---------- | ---- | ---- |
| `copilot` | `.github/`, `.vscode/` | GitHub Copilot | 検証済み |
| `claude` | `.claude/` | Claude Code | 検証済み |

> RPG-Kit は現在、CLI では **GitHub Copilot** と **Claude Code** のみをサポートしています。将来のリリースでは追加のエージェントが適応される可能性があります。

**例：**

```bash
rpgkit init my-project
rpgkit init my-project --ai claude --script sh
rpgkit init . --force
rpgkit init --here --ai copilot
rpgkit init --here --github-token $GITHUB_TOKEN
rpgkit init --here --encode
```

### `rpgkit update`

既存プロジェクト内の RPG-Kit テンプレートファイル、スクリプト、コマンド定義、MCP 設定、gitignore ルール、hooks を更新します。AI assistant は可能な場合、既存のプロジェクト設定から自動検出されます。

```bash
rpgkit update
rpgkit update --ai claude
rpgkit update --pre
rpgkit update --no-mcp
rpgkit update --github-token $GITHUB_TOKEN
```

| オプション | 説明 |
| ---------- | ---- |
| `--ai <agent>` | AI assistant。指定されていない場合は自動検出されます |
| `--script <type>` | スクリプトタイプ：`sh`（POSIX）または `ps`（PowerShell） |
| `--github-token <token>` | プライベートリポジトリまたはより高いレート制限用の GitHub token |
| `--pre` | 最新のプレリリーステンプレートをダウンロードします |
| `--no-mcp` | MCP server 設定をスキップします |
| `--skip-tls` | SSL/TLS 検証をスキップします |
| `--debug` | 詳細な診断出力を表示します |

### `rpgkit check`

必要なツールがインストールされていることを確認します。

```bash
rpgkit check
```

### `rpgkit version`

バージョンとシステム情報を表示します。

```bash
rpgkit version
```

## ワークフロー

```text
順方向：要件 → RPG → コード

 フェーズ 1：機能仕様               フェーズ 2：RPG 構築と計画                                      フェーズ 3
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
                       │ feature_edit│ feature_tree.json への任意の事前計画編集
                       └─────────────┘
                                        ╰───── rpg.json（作成 → 段階的に拡充） ─────╯
                                                                            │
                                                                            ▼
                                                                     ┌──────────┐
                                                                     │ rpg_edit │ 任意の同期 RPG + コード + dep_graph 編集
                                                                     └──────────┘

逆方向：コード → RPG

┌──────────────────┐         ┌──────────┐       ┌──────────┐
│ Existing Codebase│────────▶│  encode  │──────▶│update_rpg│
│                  │         │  (full)  │       │ (manual  │
└──────────────────┘         └──────────┘       │ fallback)│
                              rpg.json          └──────────┘
                              dep_graph.json     rpg.json / dep_graph.json
                                                  ▲
                                                  │ post-commit hook は通常インクリメンタル更新を実行

MCP Server: search_rpg / explore_rpg / get_node_detail / list_rpg_tree
```

## プロジェクト構造

`rpgkit init` を実行すると、workspace root がプロジェクトリポジトリのルートになります。RPG-Kit は、エージェントコマンド定義、ランタイムスクリプト、MCP 設定、生成データをコードと並べて追加します。

```text
my-project/
├── docs/                 # /rpgkit.feature_spec 用の任意の要件ドキュメント
├── .github/ or .claude/  # AI assistant コマンド定義と設定
├── .vscode/              # 該当する場合の Copilot/VS Code MCP 設定
└── .rpgkit/              # RPG-Kit ランタイム
    ├── scripts/          # パイプラインスクリプトとサポートパッケージ
    ├── data/             # rpg.json と dep_graph.json を含む生成アーティファクト
    ├── logs/             # ステージごとの実行ログ
    └── reports/          # 生成されたレビューおよび診断レポート
```

完全なディレクトリ構成とデータファイルリファレンスについては、[docs/project-structure.md](docs/project-structure.md) を参照してください。

## MCP 連携

`rpgkit init` は、`--no-mcp` が渡されない限り、RPG-Kit の MCP server（`rpg-tools`）を AI assistant に自動登録します。この server は `.rpgkit/data/rpg.json` を読み取り、4 つの読み取り専用ツールを公開します：

| ツール | 目的 |
| ------ | ---- |
| `search_rpg` | キーワード、名前、パス、関数、クラス、または機能でノードを検索します |
| `explore_rpg` | 開始ノードから依存関係エッジとコールチェーンエッジをたどります |
| `get_node_detail` | ノードの完全なレコードと、任意でソースコードを取得します |
| `list_rpg_tree` | リポジトリの機能アーキテクチャをツリーとしてレンダリングします |

MCP 設定は `rpgkit init` を実行したプロジェクトに限定されます。ユーザーレベルの assistant 設定は変更されません。グラフがまだ生成されていない場合、MCP ツールはエージェントに `/rpgkit.encode` を実行するよう伝える `rpg_unavailable` ヒントを返します。

MCP 登録、自動承認、hooks、初期化オプションについては、[docs/configuration.md](docs/configuration.md) を参照してください。

## トラブルシューティング

**AI assistant CLI が見つからない：**

```bash
rpgkit check
```

選択した assistant CLI をインストールして認証し、その後 `rpgkit init` または `rpgkit update` を再実行します。

**MCP ツールが `rpg_unavailable` を報告する：**

```text
/rpgkit.encode
```

MCP server は設定されていますが、`.rpgkit/data/rpg.json` はまだ作成されていません。

**インクリメンタル更新に失敗した：**

```bash
tail -n 200 .rpgkit/logs/update_rpg.log
```

次に実行します：

```text
/rpgkit.update_rpg
```

**レート制限またはプライベートリポジトリアクセスによりテンプレートのダウンロードに失敗する：**

```bash
rpgkit init my-project --github-token $GITHUB_TOKEN
# または環境変数を設定：
export GH_TOKEN=your_token
```

## ライセンス

MIT License - 詳細は [LICENSE](LICENSE) を参照してください。

## 謝辞

[GitHub Spec-Kit](https://github.com/github/spec-kit) に基づいています。
