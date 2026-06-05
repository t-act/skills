---
name: til
description: 今日のgit活動をghでリモート(GitHub)から収集し、TILエントリのドラフトを生成して追記する
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(gh *), Bash(jq *), Bash(date *), Bash(git *), Read, Edit, Write, AskUserQuestion
---

## Context

- Today (JST): !`date +%Y-%m-%d`
- Today (MMDD): !`date +%m%d`
- TIL file name (YYYY-MM): !`date +%Y-%m`
- GitHub login: !`gh api user --jq .login`

## Your task

今日のgit活動を `gh` 経由でリモート (GitHub) から収集し、TILエントリのドラフトを生成して YYYY-MM.md に追記する。
ローカルリポジトリの走査は行わない。コミット状況はすべて GitHub から取得する。

### Step 0: 現在のTILファイルを読み込む

- Read ツールで `~/Dev/til/YYYY-MM.md`（Context の TIL file name を使用）を読み込む
- ファイルが存在しない場合はスキップする（Step 4 で新規作成）

### Step 1: 今日のgit活動をリモートから収集

`gh` を使い、今日 (JST) コミットしたリポジトリと各コミットメッセージを GitHub から取得する。

まず JST 当日の境界を UTC に変換する（GitHub API の `since`/`until` は UTC）:

```bash
LOGIN="<Context の GitHub login>"
TODAY="<Context の Today (JST)>"
SINCE_UTC=$(TZ=UTC date -j -f "%Y-%m-%d %H:%M:%S %z" "${TODAY} 00:00:00 +0900" +%Y-%m-%dT%H:%M:%SZ)
UNTIL_UTC=$(TZ=UTC date -j -f "%Y-%m-%d %H:%M:%S %z" "${TODAY} 23:59:59 +0900" +%Y-%m-%dT%H:%M:%SZ)
```

**1-a. 今日コミットしたリポジトリを列挙**する。`contributionsCollection` はリアルタイムで private も含むため、これを一次ソースとする（`gh search commits` はインデックス遅延で取りこぼすため使わない）:

```bash
gh api graphql -f query='
query($from:DateTime!,$to:DateTime!){
  viewer{
    contributionsCollection(from:$from,to:$to){
      commitContributionsByRepository(maxRepositories:100){
        repository{ nameWithOwner }
        contributions{ totalCount }
      }
    }
  }
}' -F from="${TODAY}T00:00:00+09:00" -F to="${TODAY}T23:59:59+09:00" \
  --jq '.data.viewer.contributionsCollection.commitContributionsByRepository[].repository.nameWithOwner'
```

- 結果から `${LOGIN}/til`（til リポジトリ自体）は除外する。

**1-b. 各リポジトリのコミットメッセージを取得**する。1-a で得た `<owner>/<repo>` ごとに:

```bash
gh api "repos/<owner>/<repo>/commits?author=${LOGIN}&since=${SINCE_UTC}&until=${UNTIL_UTC}" \
  --jq '.[].commit.message | split("\n")[0]'
```

- `author=${LOGIN}` で自分のコミットのみに絞る。
- メッセージは1行目（サマリ）のみ使う。
- コミットが0件のリポジトリは無視する。

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
  - リポジトリ名は `<owner>/` を除いた `<repo>` 部分を使う
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

- コミット収集はすべて `gh`（リモート）から行い、ローカルリポジトリの走査はしない
- エントリは過去の記述スタイルに合わせる（簡潔に）
- 日付は `MMDD` 形式（ゼロ埋め、ハイフンなし）
- 日付境界は JST で判定する（UTC との 9 時間差に注意）
- tilリポジトリ自体のコミットは収集対象から除外する
- `contributionsCollection` はデフォルトブランチ等への「コントリビューションとして計上されたコミット」を対象とするため、まだマージされていない作業ブランチのみのコミットは収集されない場合がある
