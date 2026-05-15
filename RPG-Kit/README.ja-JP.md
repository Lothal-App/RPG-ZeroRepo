<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## コーディングエージェントに、編集する前にプランを立てさせる

コーディングエージェントはローカルな編集には強いものの、リポジトリレベルのタスクは安定した計画構造がないと失敗しがちです。要件はドリフトし、アーキテクチャ上の判断は失われ、複数ファイルにまたがる生成は一貫性を欠き、更新は隠れた依存関係を見落とすことがあります。

RPG-Kit は Claude Code と GitHub Copilot に、リポジトリレベルのコーディングのための**永続的な RPG ワークスペース**を提供します。このワークスペースは、要件・機能・アーキテクチャ・ファイル・コードエンティティ・依存関係をつなぐ **Repository Planning Graph (RPG)** を中心に構成されています。

RPG-Kit を使うと、エージェントはグラフ駆動のワークフローで作業できます:

- **Build（構築）**: 要件を RPG プランに変換し、複数ファイルからなるリポジトリを生成する。
- **Understand（理解）**: 既存のリポジトリを RPG にマッピングし、検索・探索・説明する。
- **Update（更新）**: 影響を受ける RPG ノードを特定し、編集プランを立て、コードとグラフを同時に更新する。

### ワークフローを選ぶ

| 目的 | ワークフロー | ここから始める |
|---|---|---|
| 要件から新しいリポジトリを構築する | Build ワークフロー（requirements → RPG → code） | [`クイックスタート: 新規リポジトリ`](#クイックスタート-新規リポジトリ) |
| 既存のリポジトリを理解する | Understand ワークフロー（repository → RPG → search/explore） | [`クイックスタート: 既存リポジトリ`](#クイックスタート-既存リポジトリ) |
| 既存のリポジトリを更新する | Update ワークフロー（change request → affected RPG nodes → edit plan → code/RPG update） | [`クイックスタート: 既存リポジトリ`](#クイックスタート-既存リポジトリ) |

### 詳細なパイプライン

初めて使う方は、このセクションを飛ばして下のクイックスタートから始められます。

<details>
<summary>コマンドレベルの完全なワークフロー図</summary>

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

### RPG-Kit の実例

下の図は、本リポジトリに対して生成されたグラフ可視化の一部です。`/rpgkit.encode` を実行し、`.rpgkit/data/rpg.html` を開くと完全なインタラクティブグラフを閲覧できます。

![RPG-Kit repository graph visualization](../docs/rpgkit_visualized_graph.png)

## インストール

### 前提条件

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- インストール済みで認証済みの AI コーディングエージェント CLI: [GitHub Copilot](https://docs.github.com/en/copilot) または [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

### RPG-Kit のインストール

```bash
# 永続インストール（推奨）
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
rpgkit check

# 一度きりの使用
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

## クイックスタート: 新規リポジトリ

要件から新しいコードベースを生成したい場合は、こちらの手順を使います。

> [!WARNING]
> 生成コード量が多いプロジェクトでは、`/rpgkit.design_interfaces` と `/rpgkit.code_gen` の実行に時間がかかることがあります。例として、100 個の feature でおおよそ 30 分かかります。

1. 新しいプロジェクトを初期化します:

   ```bash
   rpgkit init my-project
   cd my-project
   ```

   よく使うバリエーション:

   ```bash
   rpgkit init my-project --ai claude --script sh
   rpgkit init my-project --ai copilot
   rpgkit init my-project --github-token $GITHUB_TOKEN
   ```

2. **[任意]** 要件ドキュメントを `my-project/docs/` に配置します。

3. プロジェクトディレクトリで AI コーディングエージェントを起動します。

4. フォワードパイプラインを実行します:

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

RPG-Kit は `.rpgkit/data/rpg.json` を段階的に作成し、それを使って要件・計画成果物・生成コード・依存情報を整合した状態に保ちます。

## クイックスタート: 既存リポジトリ

すでにリポジトリがあり、AI エージェントに RPG コンテキストで理解または編集させたい場合は、こちらの手順を使います。

> [!WARNING]
> 大きめのプロジェクトでは、`rpgkit init . --encode` と `/rpgkit.encode` の実行に時間がかかることがあります。例として、200 ファイルでおおよそ 100 分かかります。

1. リポジトリのルートで RPG-Kit を初期化し、初期グラフを構築します:

   ```bash
   mkdir my-project
   cp -r existing-repo/ my-project/
   cd my-project
   rpgkit init . --encode
   ```

   空でないディレクトリでの確認プロンプトをスキップしたい場合:

   ```bash
   rpgkit init . --force --encode
   ```

2. リポジトリで AI コーディングエージェントを起動します。

3. 生成された RPG を MCP ツールおよびスラッシュコマンド経由で利用します:

   ```text
   /rpgkit.encode                                  # 必要に応じて完全な RPG を再構築
   /rpgkit.update_rpg                              # 手動の増分更新（フォールバック）
   /rpgkit.rpg_edit <edit instructions>            # グラフ認識型のコード編集
   ```

4. コミット後、RPG-Kit のフックが `.rpgkit/data/rpg.json`、`.rpgkit/data/dep_graph.json`、`.rpgkit/data/rpg.html` をコード変更に合わせて整合します。フックが失敗したりスキップされた場合は `/rpgkit.update_rpg` を実行してください。

## `rpgkit init` の後に起きること

`rpgkit init` はソースファイルを変更しません。コードのそばに、コマンド定義・ランタイムスクリプト・MCP 設定・生成されたグラフデータを追加します。

```text
my-project/
├── docs/                 # /rpgkit.feature_spec 用の任意の要件ドキュメント
├── .github/ or .claude/  # AI アシスタントのコマンド定義と設定
├── .vscode/              # 該当する場合の Copilot/VS Code MCP 設定
└── .rpgkit/              # RPG-Kit ランタイム
    ├── scripts/          # パイプラインスクリプトおよびサポートパッケージ
    ├── data/             # 生成成果物（rpg.json と dep_graph.json を含む）
    ├── logs/             # ステージごとの実行ログ
    └── reports/          # 生成時のレビュー・診断レポート
```

完全なレイアウトとデータファイルのリファレンスは [docs/project-structure.md](docs/project-structure.md) を参照してください。

## 対応プラットフォーム

| プラットフォーム      | Claude Code | GitHub Copilot | Codex |
| --------------------- | ----------- | -------------- | ----- |
| CLI 使用              | ✅          | ✅ (No MCP)    | ⌛    |
| VS Code 拡張使用      | ✅          | ✅             | ⌛    |

| スクリプト | Linux | Windows | Mac |
| ---------- | ----- | ------- | --- |
| sh         | ✅    | ⌛      | ⌛  |
| ps         | N/A   | ⌛      | ⌛  |

## ドキュメント

- [スラッシュコマンドリファレンス](docs/commands.md) — すべての `/rpgkit.*` コマンドの入力・出力・例。
- [CLI リファレンス](docs/cli-reference.md) — `rpgkit init`、`rpgkit update`、`rpgkit check`、`rpgkit version` とすべてのオプション。
- [設定](docs/configuration.md) — AI アシスタントのセットアップ、MCP 登録、フック、自動承認、およびトラブルシューティング。
- [プロジェクト構造](docs/project-structure.md) — RPG-Kit が作成するファイルとディレクトリ。

## 今後の機能

- **よりシンプルな生成コマンド:** 現在の多段階の生成フローを、`/rpgkit.generate_repo`、`/rpgkit.generate_feature`、`/rpgkit.plan` などのより少ないコマンドにまとめます。
- **多言語サポート:** Go、C++、Rust、JavaScript/TypeScript などのサポートを追加します。
- **より多くのプラットフォーム連携:** さまざまなシステム上の異なる AI コーディングエージェントについて、CLI と VS Code 拡張ワークフローを横断して RPG-Kit をサポートします。

## トラブルシューティング

**AI アシスタント CLI が見つからない:** `rpgkit check` を実行し、選択したアシスタント CLI をインストールおよび認証し、`rpgkit init` または `rpgkit update` を再実行してください。

**MCP ツールが `rpg_unavailable` を報告する:** `/rpgkit.encode` を実行して `.rpgkit/data/rpg.json` を作成してください。

**増分更新が失敗する:** `.rpgkit/logs/update_rpg.log` を確認し、`/rpgkit.update_rpg` を実行してください。

**レート制限またはプライベートリポジトリのアクセス権でテンプレートのダウンロードに失敗する:** `--github-token $GITHUB_TOKEN` を渡すか、`GH_TOKEN` / `GITHUB_TOKEN` を設定してください。

## ライセンス

MIT License — 詳細は [LICENSE](LICENSE) を参照してください。

## 謝辞

[GitHub Spec-Kit](https://github.com/github/spec-kit) を基にしています。
