---
name: study-commit
description: 技術書やUdemyなどの学習コミットを自動ステージング・差分分析し、適切なprefixとメッセージを提案してコミットする
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(git add *), Bash(git status *), Bash(git diff *), Bash(git commit *), Bash(git log *), AskUserQuestion, Read, Edit
---

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Recent commits (current repo): !`git log --oneline -50`
- Current branch: !`git branch --show-current`
## Your task

技術書やオンラインコースの学習時に、変更内容を分析して適切なコミットメッセージを提案し、ユーザーの承認後にコミットを実行します。

### Step 0: 過去のコミット例を読み込む

- Read ツールで `~/.claude/skills/study-commit/examples.md` を読み込む
- ファイルが存在しない場合やエントリが空の場合はスキップする
- 読み込んだ例をコミットメッセージ生成の参考にする

### Step 1: 変更の分析とグループ化

- git status と diff から変更ファイル・変更内容を把握する
- 変更がない場合はその旨を伝えて終了する
- 変更ファイルを論理的な単位（1セクション = 1コミット）にグループ化する

グループ化の指針:
- 同じ機能・トピックに関連するファイルを1グループにまとめる（例: seq2seq.py と それが import するモジュールを同時に新規作成した場合は同一グループ）
- 独立した実装は別グループに分ける（例: seq2seq.py と encoder.py がそれぞれ独立したセクションの成果物なら別グループ）
- 判断に迷う場合はユーザーに確認する
- グループが1つしかない場合はそのまま進める

### Step 2: prefix の判定

以下の prefix から、変更内容に最も適切なものを判定する:

| prefix | 用途 | 例 |
|---|---|---|
| `feat` | 書籍のコードを写経・実装、演習問題の解答 | `feat: ch06/seq2seq.pyを実装` |
| `fix` | 実装中のバグを修正 | `fix: Encoderの隠れ状態の受け渡しを修正` |
| `refactor` | 既存コードのリファクタリング | `refactor: Trainerクラスの共通処理を抽出` |
| `docs` | メモ・ノートの追加 | `docs: Attention機構の仕組みをメモ` |
| `exp` | 自分なりの実験・改変 | `exp: dropout率を変えて精度を比較` |
| `copy` | 技術書が提供しているソースからコピー | `copy: ch06/dataset.pyを追加` |

判定の指針:
- diff 内のコメントに `@copied` マーカーが含まれている → `copy`（例: `# @copied`, `// @copied`）
- 新規ファイルで、段階的に実装している痕跡がある → `feat`
- 既存ファイルのバグ修正 → `fix`
- 既存ファイルの構造改善（動作変更なし）→ `refactor`
- .md, .txt, メモ系ファイルの追加・変更 → `docs`
- 既存実装に対するパラメータ変更や独自の試行 → `exp`
- 判断に迷う場合はユーザーに確認する

### Step 3: グループごとの繰り返し処理

各グループに対して以下を順番に実行する（全グループが完了するまで繰り返す）:

#### 3a: 対象ファイルの提示

- 現在のグループに含まれるファイル一覧をユーザーに提示する
- 「グループ N/M」の形式で進捗を表示する

#### 3b: コミットメッセージの生成と提案

- 過去のコミット履歴のスタイルに合わせる
- 日本語で記述（prefix のみ英語）
- 1行で完結（本文なし）
- 3つの候補を生成し、AskUserQuestion でユーザーに提示する
- 各候補は異なる詳細度や表現を提供する

#### 3c: コミット実行

- ユーザーが候補を選択（またはカスタムメッセージを入力）したら、対象グループのファイルのみを `git add <files>` でステージング
- 選択されたメッセージでコミットを実行する
- 次のグループへ進む

### Step 4: examples.md の更新

全グループのコミットが完了した後、選択されたコミットメッセージを `~/.claude/skills/study-commit/examples.md` に追記する。

- Read ツールで examples.md を読み込む
- 各コミットメッセージを `- <message>` の形式で末尾に追記する（Edit ツールを使用）
- 追記後、エントリ数が 30 件を超えている場合は、古いもの（上部）から削除して 30 件に収める
- examples.md のヘッダー（`# Study Commit Examples` と `<!-- ... -->` コメント行）は維持する

## Constraint

- Claude co-authorship フッターは不要
- メッセージは日本語（prefix のみ英語）
- 1行で完結させる（本文不要）
- ステージングはグループ単位で行う（`git add -A` は使わない）
