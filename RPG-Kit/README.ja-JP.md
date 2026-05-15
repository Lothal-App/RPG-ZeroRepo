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

- **順方向（要件 → RPG → コード）：** AI コーディングエージェントによる多段階パイプラインを通じて、高レベルの要件をテスト済みで構造化されたリポジトリに変換します。[クイックスタートはこちら](#quick-start-new-repository)
- **逆方向（コード → RPG）：** 既存のコードベースを RPG グラフにエンコードし、AI 支援の検索、探索、インクリメンタル更新に利用します。[クイックスタートはこちら](#quick-start-existing-repository)
- **外科的編集（指示 → RPG + コード）：** コード、RPG、依存グラフの同期を保ちながら、対象を絞った変更を適用します。[クイックスタートはこちら](#quick-start-existing-repository)

## インストール

### 前提条件

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- インストール済みで認証済みの AI コーディングエージェント CLI：[GitHub Copilot](https://docs.github.com/en/copilot) または [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

### RPG-Kit をインストールする

```bash
# 永続インストール（推奨）
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
rpgkit check

# 一回限りの使用
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

<a id="quick-start-new-repository"></a>
## クイックスタート：新規リポジトリ

RPG-Kit に要件を新しいコードベースへ変換させたい場合は、この手順を使用します。

1. 新しいプロジェクトを初期化します：

   ```bash
   rpgkit init my-project
   cd my-project
   ```

   一般的なバリエーション：

   ```bash
   rpgkit init my-project --ai claude --script sh
   rpgkit init my-project --ai copilot
   rpgkit init my-project --github-token $GITHUB_TOKEN
   ```

2. **[任意]** 要件ドキュメントを `my-project/docs/` に配置します。

3. プロジェクトディレクトリで AI コーディングエージェントを起動します。

4. 順方向パイプラインを実行します：

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

RPG-Kit は `.rpgkit/data/rpg.json` を段階的に作成し、それを使用して要件、計画成果物、生成されたコード、依存関係情報の整合性を保ちます。

<a id="quick-start-existing-repository"></a>
## クイックスタート：既存リポジトリ

すでにリポジトリがあり、AI エージェントに RPG コンテキストを使って理解または編集させたい場合は、この手順を使用します。

1. リポジトリルートで RPG-Kit を初期化し、初期グラフを構築します：

   ```bash
   mkdir my-project
   cp -r existing-repo/ my-project/
   cd my-project
   rpgkit init . --encode
   ```

   空でないディレクトリの確認プロンプトをスキップしたい場合：

   ```bash
   rpgkit init . --force --encode
   ```

2. リポジトリ内で AI コーディングエージェントを起動します。

3. MCP ツールとスラッシュコマンドを通じて、生成された RPG を使用します：

   ```text
   /rpgkit.encode                                  # 必要に応じて完全な RPG を再構築
   /rpgkit.update_rpg                              # 手動インクリメンタル更新のフォールバック
   /rpgkit.rpg_edit <edit instructions>            # グラフ認識型コード編集
   ```

4. コミット後、RPG-Kit hooks は `.rpgkit/data/rpg.json`、`.rpgkit/data/dep_graph.json`、`.rpgkit/data/rpg.html` をコード変更と整合させます。hook が失敗またはスキップされた場合は、`/rpgkit.update_rpg` を実行してください。

## 追加されるもの

`rpgkit init` の実行後も、workspace root はプロジェクトリポジトリのルートのままです。RPG-Kit は、コマンド定義、ランタイムスクリプト、MCP 設定、生成されたグラフデータをコードと並べて追加します。

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

完全なレイアウトとデータファイルリファレンスについては、[docs/project-structure.md](docs/project-structure.md) を参照してください。

## サポートされるプラットフォーム

| プラットフォーム        | Claude Code | GitHub Copilot | Codex |
| ----------------------- | ----------- | -------------- | ----- |
| CLI 使用                | ✅          | ✅(MCP なし)   | ⌛    |
| VS Code 拡張の使用      | ✅          | ✅             | ⌛    |

| スクリプト | Linux | Windows | Mac |
| ---------- | ----- | ------- | --- |
| sh         | ✅    | ⌛      | ⌛  |
| ps         | N/A   | ⌛      | ⌛  |

## ドキュメント

- [スラッシュコマンドリファレンス](docs/commands.md) — すべての `/rpgkit.*` コマンド、入力、出力、例。
- [CLI リファレンス](docs/cli-reference.md) — `rpgkit init`、`rpgkit update`、`rpgkit check`、`rpgkit version`、およびすべてのオプション。
- [設定](docs/configuration.md) — AI assistant のセットアップ、MCP 登録、hooks、自動承認、トラブルシューティング。
- [プロジェクト構造](docs/project-structure.md) — RPG-Kit が作成するファイルとディレクトリ。

## 今後の機能

- **よりシンプルなデコーダーコマンド：** 現在のデコーダーフローをより少ないコマンドに統合します。これには、エンドツーエンドのリポジトリ生成用の `/rpgkit.generate_repo`、および機能生成と RPG 計画用の `/rpgkit.generate_feature` と `/rpgkit.plan` が含まれます。
- **多言語サポート：** Go、C++、Rust、JavaScript/TypeScript などのサポートを追加します。
- **より多くのプラットフォーム統合：** さまざまなシステム上で、異なる AI コーディングエージェント向けに CLI と VS Code 拡張ワークフローで RPG-Kit をサポートします。

## トラブルシューティング

**AI assistant CLI が見つからない：** `rpgkit check` を実行し、選択した assistant CLI をインストールして認証したうえで、`rpgkit init` または `rpgkit update` を再実行します。

**MCP ツールが `rpg_unavailable` を報告する：** `/rpgkit.encode` を実行して `.rpgkit/data/rpg.json` を作成します。

**インクリメンタル更新に失敗した：** `.rpgkit/logs/update_rpg.log` を確認し、その後 `/rpgkit.update_rpg` を実行します。

**レート制限またはプライベートリポジトリアクセスによりテンプレートのダウンロードに失敗する：** `--github-token $GITHUB_TOKEN` を渡すか、`GH_TOKEN` / `GITHUB_TOKEN` を設定します。

## ライセンス

MIT License - 詳細は [LICENSE](LICENSE) を参照してください。

## 謝辞

[GitHub Spec-Kit](https://github.com/github/spec-kit) に基づいています。
