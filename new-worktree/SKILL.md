---
name: new-worktree
description: 作業内容からブランチ名(feature/<内容> 形式)を3案提示して選ばせ、最新の origin/<デフォルトブランチ> から .claude/worktrees/ にワークツリーを切り、.env 等をコピーしてセッションごと移動する。ワークツリーで新しい作業を始めるときの初期化用
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(git *), Bash(mkdir *), Bash(cp *), Bash(echo *), AskUserQuestion, EnterWorktree
---

## Context

- 作業内容 (引数): $ARGUMENTS
- 現在地: !`git rev-parse --show-toplevel 2>/dev/null || echo "(git リポジトリではない)"`
- git 共通ディレクトリ: !`git rev-parse --path-format=absolute --git-common-dir 2>/dev/null`
- 現在のブランチ: !`git branch --show-current 2>/dev/null`
- リモートのデフォルトブランチ: !`git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null || echo "(origin/HEAD 未設定)"`
- 既存のワークツリー: !`git worktree list 2>/dev/null`
- 最近のブランチ (命名傾向の参考): !`git for-each-ref --sort=-committerdate --count=15 --format='%(refname:short)' refs/heads refs/remotes 2>/dev/null`

## Your task

ワークツリーで新しい作業を始めるための初期化をまとめて行う。
流れは **最新化 → ブランチ名を決める → ワークツリーを切る → 設定ファイルを持ち込む → セッションを移す** の順。

### Step 0: 前提の確認

- Context の「現在地」が git リポジトリでなければ、その旨を伝えて終了する。
- メインリポジトリのルート（以下 `ROOT`）は、Context の「git 共通ディレクトリ」の親ディレクトリとする。
  `--show-toplevel` を使わないのは、ワークツリーの中から呼ばれるとワークツリー自身のパスが返るため。
- 作業内容 (引数) が空なら、何をするのかを一言で聞き、返答を待ってから次に進む。

### Step 1: 最新化

分岐元を最新にし、あわせてリモートのブランチ名を Step 2 の衝突チェックに使えるようにする。

```bash
git -C "$ROOT" fetch origin
```

- 分岐元 `BASE` は Context の「リモートのデフォルトブランチ」（例: `origin/main`）。
- 「origin/HEAD 未設定」なら、次で設定してから取り直す。
  ```bash
  git -C "$ROOT" remote set-head origin --auto
  git -C "$ROOT" symbolic-ref --short refs/remotes/origin/HEAD
  ```
- origin リモートが無い、または fetch に失敗した場合は、ローカルの `main`（無ければ `master`）を分岐元にしてよいか AskUserQuestion で確認する。古い可能性がある旨を添える。

### Step 2: ブランチ名を決める

作業内容から、次の規則でブランチ名を3案作り、AskUserQuestion で選ばせる（「Other」で自由入力もできる）。

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
git -C "$ROOT" rev-parse --verify --quiet "refs/heads/<branch>"
git -C "$ROOT" rev-parse --verify --quiet "refs/remotes/origin/<branch>"
```

どちらかが SHA を返したら使用済み。ワークツリーのディレクトリ（Step 3）が既にある名前も外す。

「Other」で入力された名前は、`git check-ref-format --branch "<branch>"` と衝突チェックを通してから使う。
種別が無いなど形式から外れていれば、そのまま使うか直すかを確認する。

### Step 3: ワークツリーを作る

ディレクトリ名は、ブランチ名の `/` を `-` に置き換えたもの（`feature/login-form` → `feature-login-form`）。

まず、メインリポジトリの `git status` にワークツリーが未追跡として出ないよう、`.claude/worktrees/` が ignore されているか確かめる。

```bash
git -C "$ROOT" check-ignore -q .claude/worktrees/probe
```

終了コードが 0 以外なら、ローカルの exclude に追記する。

```bash
echo '.claude/worktrees/' >> "<git 共通ディレクトリ>/info/exclude"
```

`.gitignore` に書かないのは、コミット対象の変更を初期化のついでに混ぜないため。

続けてブランチとワークツリーを同時に作る。

```bash
git -C "$ROOT" worktree add --no-track -b "<branch>" "$ROOT/.claude/worktrees/<dir>" "$BASE"
```

`--no-track` を付けるのは、`origin/main` から切ると upstream が `origin/main` になり、`git status` が main との差分を出し続けるうえ、`push.default` の設定によっては `git push` が main へ向かうため。
upstream は初回 push で `git push -u origin <branch>` として張る。

### Step 4: git 管理外の設定ファイルをコピー

ワークツリーには追跡ファイルしか展開されないため、ignore された設定ファイルをメインリポジトリから持ち込む。

```bash
git -C "$ROOT" ls-files --others --ignored --exclude-standard --directory
```

- 出力のうち、ファイル名が `.env` / `.env.*` / `.dev.vars` / `.dev.vars.*` に当たるものを対象にする。
- `--directory` を付けるのは、`node_modules/` のような ignore されたディレクトリの中身を列挙させないため。そのぶん ignore されたディレクトリ内の `.env` は対象外になる。
- 対象ごとに、相対パスを保ったまま複製する。
  ```bash
  mkdir -p "$ROOT/.claude/worktrees/<dir>/<親ディレクトリ>"   # ルート直下なら不要
  cp -p "$ROOT/<file>" "$ROOT/.claude/worktrees/<dir>/<file>"
  ```
- 中身は表示しない（秘密情報を含むため）。コピーしたパスだけを控えておく。
- 対象が無ければ何もしない。

### Step 5: セッションを移す

EnterWorktree に、作ったワークツリーの絶対パスを `path` として渡す。

`name` で作らせないのは、`name` だとブランチ名を Step 2 で決めた名前にできないため。

失敗した場合は、エラー内容とワークツリーのパスを伝え、そのディレクトリで `claude` を起動し直せば作業を始められると案内する。

### Step 6: 結果の報告

次を簡潔に表示する。

- ブランチ名と分岐元（例: `feature/login-form` ← `origin/main` @ `a1b2c3d`）
- ワークツリーのパス
- コピーしたファイル（無ければ「なし」）
- `info/exclude` に追記したかどうか
- 後片付けの方法。EnterWorktree の `path` で入ったワークツリーは ExitWorktree が削除しないため、手で消す。
  ```bash
  git worktree remove .claude/worktrees/<dir>
  git branch -d <branch>
  ```

## Constraint

- ブランチ名はユーザーの選択を経ずに確定しない。
- 分岐元は origin のデフォルトブランチの最新とする。ユーザーが明示しない限り、現在のブランチから切らない。
- ワークツリーは必ず `$ROOT/.claude/worktrees/` 配下に作る。
- メインリポジトリの作業ツリーには触らない（stash・checkout・reset をしない）。
- 依存パッケージのインストールと push は行わない。
- `.env` 等の中身を出力しない。
