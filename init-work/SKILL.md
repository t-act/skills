---
name: init-work
description: claude -w で起動したワークツリーのセッションで、作業内容からブランチ名(feature/<内容> 形式)を3案提示して選ばせ、自動で付いた worktree-<名前> ブランチを選んだ名前に改名する。分岐元が古ければ origin のデフォルトブランチまで fast-forward する
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(git *), Bash(ls *), AskUserQuestion
---

## Context

- 作業内容 (引数): $ARGUMENTS
- 現在のブランチ: !`git branch --show-current 2>/dev/null`
- git ディレクトリ: !`git rev-parse --path-format=absolute --git-dir 2>/dev/null || echo "(git リポジトリではない)"`
- git 共通ディレクトリ: !`git rev-parse --path-format=absolute --git-common-dir 2>/dev/null`
- リモートのデフォルトブランチ: !`git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null || echo "(origin/HEAD 未設定)"`
- 作業ツリーの状態: !`git status --short 2>/dev/null`
- 最近のブランチ (命名傾向の参考): !`git for-each-ref --sort=-committerdate --count=15 --format='%(refname:short)' refs/heads refs/remotes 2>/dev/null`

## Your task

`claude -w <名前>` で起動したセッションで呼ばれる前提で動く。
ワークツリーは Claude Code が `.claude/worktrees/<名前>` に作成済みで、ブランチには `worktree-<名前>` が自動で付いている。
このブランチを作業内容に合った名前に付け替え、分岐元を最新にする。

### Step 0: 前提の確認

- Context の「git ディレクトリ」と「git 共通ディレクトリ」が同じなら、ワークツリーではなくメインの作業ツリーにいる。`claude -w <名前>` で起動し直すよう伝えて終了する。
- 現在のブランチが空（detached HEAD）なら、その旨を伝えて終了する。
- 現在のブランチが `worktree-` で始まらなければ、すでに名前を付けてある可能性がある。改名してよいか AskUserQuestion で確認する。
- 作業内容 (引数) が空なら、何をするのかを一言で聞き、返答を待ってから次に進む。

以下、メインリポジトリのルート（「git 共通ディレクトリ」の親）を `ROOT`、分岐元を `BASE`（「リモートのデフォルトブランチ」、例: `origin/main`）と呼ぶ。

### Step 1: 最新化

```bash
git fetch origin
```

- 「origin/HEAD 未設定」なら、`git remote set-head origin --auto` を実行してから `git symbolic-ref --short refs/remotes/origin/HEAD` で `BASE` を取り直す。
- origin が無い、または fetch に失敗した場合は最新化を飛ばし、その旨を報告に含める。

次の条件をすべて満たすときだけ、`BASE` まで fast-forward する。

- `git status --porcelain` が何も出力しない
- `git merge-base --is-ancestor HEAD "$BASE"` が成功する（HEAD が `BASE` に含まれ、自分のコミットがない）
- `git rev-list --count "HEAD..$BASE"` が 1 以上

```bash
git merge --ff-only "$BASE"
```

条件を満たさなければ HEAD は動かさない。
reset や rebase で揃えないのは、`worktree.baseRef` を `head` にしてデフォルトブランチ以外から切った場合などを壊さないため。

### Step 2: ブランチ名を決める

作業内容から、次の規則でブランチ名を3案作り、AskUserQuestion で選ばせる（「Other」で自由入力もできる）。
ワークツリー名は `claude -w` で決まっているので扱わない。

**形式**: `<種別>/<内容>`

| 種別 | 使いどころ |
| --- | --- |
| `feature` | 機能の追加 |
| `fix` | 不具合の修正 |
| `update` | 既存機能・依存・設定の更新 |
| `refactor` | 挙動を変えない整理 |
| `docs` | 文書のみ |
| `test` | テストのみ |
| `chore` | その他の雑務 |
| `exp` | 捨てる前提の試行錯誤 |

**`<内容>` の規則**:

- 英小文字・数字・ハイフンのみ（kebab-case）。2〜4語、ブランチ名全体で40文字以内を目安にする。
- 何をするかが伝わる名詞句にする（`login-form`、`ws-reconnect`）。`fix/bug` や `feature/update` のような中身のない名前にしない。
- Context の「最近のブランチ」に独自の傾向があれば合わせる。

**3案の出し分け**: 種別の解釈（`feature` か `update` か）や粒度（`login-form` / `login-form-validation`）など、観点を変える。同じ案の言い換えを3つ並べない。

**衝突チェック**: 提示する前に各案を検査し、使われている名前は外して作り直す。

```bash
git rev-parse --verify --quiet "refs/heads/<branch>"
git rev-parse --verify --quiet "refs/remotes/origin/<branch>"
```

どちらかが SHA を返したら使用済み。

「Other」で入力された名前は、`git check-ref-format --branch "<branch>"` と衝突チェックを通してから使う。
種別が無いなど形式から外れていれば、そのまま使うか直すかを確認する。

### Step 3: 改名する

```bash
git branch -m "<branch>"
```

新しいブランチを切らずに改名するのは、切ると `worktree-<名前>` が使われないブランチとして残るため。

改名後に upstream を確認する。

```bash
git rev-parse --abbrev-ref '@{u}'
```

`BASE` が表示されたら外す。

```bash
git branch --unset-upstream
```

`origin/main` を upstream のままにしないのは、`git status` が main との差分を出し続けるうえ、`push.default` の設定によっては `git push` が main へ向かうため。
upstream は初回 push で `git push -u origin <branch>` として張る。

### Step 4: .worktreeinclude の確認

`.env` などの持ち込みは Claude Code の `.worktreeinclude` に任せ、この skill ではコピーしない。
設定漏れに気づけるよう、次を確かめる。

```bash
ls "$ROOT/.worktreeinclude"
git -C "$ROOT" ls-files --others --ignored --exclude-standard --directory
```

`.worktreeinclude` が無く、2つ目の出力にファイル名が `.env` / `.env.*` / `.dev.vars` / `.dev.vars.*` のものがあれば、報告で次の設定を案内する。ファイルは作らない。

```gitignore
# $ROOT/.worktreeinclude（gitignore と同じ書式）
.env
.env.*
.dev.vars
```

設定しても反映されるのは次回の `claude -w` からで、今のワークツリーには入らないことも添える。

### Step 5: 結果の報告

次を簡潔に表示する。

- ブランチ名の変更（例: `worktree-login` → `feature/login-form`）
- 分岐元（例: `origin/main` @ `a1b2c3d`）と、fast-forward したか。飛ばしたならその理由
- upstream を外したかどうか
- `.worktreeinclude` の案内（該当するときだけ）
- セッション終了時に「remove」を選んでも、Claude Code は元の名前 `worktree-<名前>` でブランチを消そうとするため、改名後のブランチは残る。不要なら `git branch -D <branch>` で消す。

## Constraint

- ブランチ名はユーザーの選択を経ずに確定しない。
- メインの作業ツリーにいるときは改名しない。
- ワークツリーの作成・移動・削除はしない（`claude -w` の役割）。
- fast-forward 以外で HEAD を動かさない（reset・rebase をしない）。
- `.env` などのコピーや `.worktreeinclude` の作成はしない。案内にとどめる。
- push しない。
