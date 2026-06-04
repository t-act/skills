# Claude Code Skills

個人用の [Claude Code](https://docs.claude.com/en/docs/claude-code) スキル集。日々の開発でよく繰り返す作業（コミット、PR作成、学習記録など）をスラッシュコマンドとして定義し、`~/.claude/skills/` 配下に配置して利用する。

## 収録スキル

| コマンド | 概要 |
| --- | --- |
| `/commit` | 変更内容を分析し、コミットメッセージ候補を3つ提示して選択・コミット |
| `/create-pr` | main との差分から PR 説明文を自動生成し、GitHub PR を作成 |
| `/paper-summary` | 論文 (PDF/md) を読み込み、タイトル名ディレクトリに日本語要約 HTML を生成（[README](./paper-summary/README.md)） |
| `/readme` | マニフェストから技術スタックを検出し、README を対話的に作成・更新 |
| `/study-commit` | 技術書・Udemy などの学習コミットを自動ステージ＆prefix 付きでコミット |
| `/til` | 複数リポジトリから今日の git 活動を集約し、TIL エントリを生成 |

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
├── create-pr/                  # PR作成
├── paper-summary/              # 論文の日本語要約 HTML 生成
├── readme/                     # README 生成（本スキル）
├── study-commit/               # 学習用コミット
└── til/                        # Today I Learned 集約
```

各ディレクトリに `SKILL.md` があり、frontmatter にメタ情報（`name`, `description`, `allowed-tools` など）、本文に実行手順が記載されている。
