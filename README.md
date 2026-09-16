# Claude Code Skills

個人用の [Claude Code](https://docs.claude.com/en/docs/claude-code) スキル集。日々の開発でよく繰り返す作業（コミット、学習記録など）と、HTML や Python コード、日本語の技術文書を書くときの規約をスキルとして定義し、`~/.claude/skills/` 配下に配置して利用する。

## 収録スキル

### スラッシュコマンド

| コマンド | 概要 |
| --- | --- |
| `/commit` | 変更内容を分析し、コミットメッセージ候補を3つ提示して選択・コミット |
| `/init-work` | `claude -w` で起動したワークツリーで、作業内容からブランチ名を3案提示し、選んだ名前にブランチを改名 |
| `/paper-summary` | 論文 (PDF/md) を読み込み、タイトル名ディレクトリに日本語要約 HTML を生成（[README](./paper-summary/README.md)） |
| `/readme` | マニフェストから技術スタックを検出し、README を対話的に作成・更新 |
| `/study-commit` | 技術書・Udemy などの学習コミットを自動ステージ＆prefix 付きでコミット |
| `/til` | 複数リポジトリから今日の git 活動を集約し、TIL エントリを生成 |

### 規約・ガイド

該当する作業（HTML の出力、Python コードの記述、日本語の技術文書の執筆）で Claude が自動的に参照する。`/` から呼び出すこともできる。

| スキル | 概要 |
| --- | --- |
| `html-style-guide` | 単一 HTML を出力するときの統一スタイルガイド。Claude 風のデザイントークンとコンポーネント（表、コード差分など）をライト/ダーク両対応で定義（見本: [preview.html](./html-style-guide/preview.html)） |
| `jp-tech-writing` | 日本語の技術文書・書籍原稿の文章規範。パラグラフライティング、論証の厳密さ、LLM っぽい表現の禁止、冗長の排除、Design doc と PR 説明文の結論先出しを定める |
| `python-code-quality` | Python コードの品質規約。意味のある値の定数化、docstring とコメントに Why not を書くこと、ruff の実行を定める |

## インストール

Claude Code はユーザースキルを `~/.claude/skills/<name>/SKILL.md` から読み込むため、本リポジトリを同ディレクトリへ配置する。

```bash
git clone <this-repo> ~/.claude/skills
```

既に `~/.claude/skills` が存在する場合は中身をマージする。各スキルは独立ディレクトリなので、必要なものだけコピーしてもよい。

インストール後、Claude Code セッション内で `/` を入力するとコマンド候補に各スキルが現れる。

## ディレクトリ構成

```
.
├── commit/                     # コミットメッセージ生成
├── html-style-guide/           # HTML 出力の統一スタイルガイド
├── init-work/                  # ワークツリーのブランチ名決定
├── jp-tech-writing/            # 日本語技術文書の文章規範
├── paper-summary/              # 論文の日本語要約 HTML 生成
├── python-code-quality/        # Python コード品質規約
├── readme/                     # README 生成（本スキル）
├── study-commit/               # 学習用コミット
└── til/                        # Today I Learned 集約
```

各ディレクトリに `SKILL.md` があり、frontmatter にメタ情報（`name`, `description`, `allowed-tools` など）、本文に実行手順や規約が記載されている。
