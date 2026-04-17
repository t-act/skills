---
name: readme
description: プロジェクトのREADME.mdを対話的に作成・更新する。マニフェストから技術スタックを自動検出し、diff形式のプレビューで承認を得てから書き込む
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(ls *), Bash(cat *), Bash(diff *), Bash(find *), Read, Write, Edit, Glob, Grep, AskUserQuestion
---

## Context

- Working directory: !`pwd`
- 既存README有無: !`ls README.md 2>/dev/null || echo "NOT_FOUND"`
- マニフェストファイル検出: !`ls package.json pyproject.toml requirements.txt Cargo.toml go.mod Gemfile composer.json 2>/dev/null`
- ディレクトリ構成（2階層）: !`find . -maxdepth 2 -type d -not -path '*/node_modules*' -not -path '*/.git*' -not -path '*/dist*' -not -path '*/build*' -not -path '*/.next*' -not -path '*/__pycache__*' -not -path '*/target*' | sort`

## Your task

README.mdを作成または差分更新します。既存があれば言語を判定し、プロジェクト情報を自動検出したうえで、ユーザーに確認を取りながら生成します。

### 1. 既存README判定

- `README.md` が存在すれば Read で読み込み、使用言語（日本語/英語）を判定
- 無ければ新規作成モード（言語は日本語をデフォルト）

### 2. プロジェクト情報の自動検出

検出されたマニフェストを Read して以下を抽出：

| ファイル | 抽出項目 |
| --- | --- |
| `package.json` | name, description, scripts, dependencies, devDependencies |
| `pyproject.toml` / `requirements.txt` | プロジェクト名, 依存 |
| `Cargo.toml` | name, dependencies |
| `go.mod` | module名, Go version |
| `Gemfile` | gem 一覧 |
| `composer.json` | name, require |

さらに、Glob で主要ディレクトリ（`src/**`, `app/**`, `lib/**`, `components/**` など）の構成を確認する。

### 3. ユーザーへのヒアリング

AskUserQuestion を使用：

- **必須**: プロジェクト概要・目的（マニフェストの description だけでは不十分な場合）
- **任意**: 推測困難で README に含めたほうが価値がある項目がある場合のみ追加質問
  - 例: ターゲットユーザー、主要機能、公開予定、ライセンスなど
  - 不要だと判断できるなら聞かない

### 4. コンテンツ生成

以下のセクションを含む README を生成：

1. **プロジェクト概要・目的** — タイトル（H1）と導入文
2. **技術スタック** — 自動検出した言語・フレームワーク・主要ライブラリ
3. **セットアップ・起動手順** — マニフェストのスクリプト/標準コマンドから導出
4. **ディレクトリ構成** — 主要ディレクトリのみ（ツリー形式）

出力言語は既存README追従、新規は日本語。

### 5. diff プレビューと承認

- **新規作成時**: 生成した全文を提示したうえで AskUserQuestion で承認を取る
- **更新時**: 既存内容との diff（追加行 `+` / 削除行 `-`）を提示し、AskUserQuestion で承認を取る
  - 不足セクションは追加
  - 既存セクションの自動検出項目（スタック・スクリプト・構成）は最新化
  - 古くなった記述（削除済みスクリプト/依存）は削除
  - 手書き説明文も更新対象（保護しない）

### 6. 書き込み

- 新規: Write で `README.md` を作成
- 更新: Edit で差分を反映（全体書き換えが妥当な場合は Write）

## Constraint

- co-authorship フッターは付与しない
- プレビュー・承認なしで書き込まない
- 検出できなかった情報を勝手に創作しない（不明ならヒアリング、それでも不明ならセクションを省略）
- 既存READMEの言語に合わせる。新規は日本語
