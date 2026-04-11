---
name: til
description: 今日のgit活動を複数リポジトリから収集し、TILエントリのドラフトを生成して追記する
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(git *), Bash(ls *), Bash(date *), Bash(for *), Read, Edit, Write, AskUserQuestion
---

## Context

- Today (JST): !`date +%Y-%m-%d`
- Today (MMDD): !`date +%m%d`
- TIL file name (YYYY-MM): !`date +%Y-%m`

## Your task

今日のgit活動を複数リポジトリから収集し、TILエントリのドラフトを生成してYYYY-MM.mdに追記する。

### Step 0: 現在のTILファイルを読み込む

- Read ツールで `~/Dev/til/YYYY-MM.md`（Context の TIL file name を使用）を読み込む
- ファイルが存在しない場合はスキップする（Step 4 で新規作成）

### Step 1: 今日のgit活動を収集

以下のディレクトリ配下にある全gitリポジトリから、今日のコミットを収集する:

- `~/Dev/`
- `~/Lab/`
- `~/Exp/`

収集方法:
- 各ディレクトリの直下にあるサブディレクトリを走査する
- `.git` が存在するディレクトリのみ対象とする
- `git log --oneline --since="$(date +%Y-%m-%d) 00:00" --until="$(date +%Y-%m-%d) 23:59"` で当日コミットを取得する
- tilリポジトリ自体(`~/Dev/til`)は除外する

### Step 2: エントリのドラフト生成

収集結果をもとに、TILのエントリ形式でドラフトを生成する。

フォーマット:
```
MMDD
- 学習内容1
- 学習内容2
```

ドラフト生成の指針:
- 「リポジトリ名（トピックキーワード）」の形式で書く
  - 例: `deep-learning-2（Seq2Seq、PeekyDecoder実装）`、`ts入門（型ガード、ジェネリクス）`
  - コミットメッセージから章・テーマ・キーワードを抽出してカッコ内に要約する
  - カッコ内は短く、1〜3個のキーワード程度にする
- 同一リポジトリへの複数コミットは1行にまとめる
- コミットが0件の場合は `- No` とする

### Step 3: ユーザー確認

- 収集したリポジトリ別のコミット一覧を表示する
- 生成したドラフトを表示する
- AskUserQuestion で以下を同時に確認する:
  - ドラフトの内容: 「このまま追記」「編集してから追記」「キャンセル」
  - コミット＆プッシュ: 「追記後にコミット＆プッシュする」「追記のみ（コミットしない）」

### Step 4: TILファイルへの追記とコミット

- 対象ファイル: `~/Dev/til/YYYY-MM.md`（当月）
- ファイルが存在しない場合は新規作成する
- 今日の日付のエントリが既に存在する場合:
  - 既存エントリを上書きする
- Edit ツールで末尾に追記する
- Step 3 でコミット＆プッシュを選択した場合:
  - `update: YYYY-MM-DD` 形式でコミットする（既存の慣習に従う）
  - コミット後に `git push` でリモートにプッシュする

## Constraint

- エントリは過去の記述スタイルに合わせる（簡潔に）
- 日付は `MMDD` 形式（ゼロ埋め、ハイフンなし）
- tilリポジトリ自体のコミットは収集対象から除外する
