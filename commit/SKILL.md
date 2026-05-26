---
name: commit
description: 変更内容を論理的な単位に分割し、複数のコミットを順次作成する
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(git add *), Bash(git status *), Bash(git commit *), Bash(git diff *), Bash(git reset *), Bash(git log *), AskUserQuestion
---

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD 2>/dev/null || { echo "(no HEAD yet — showing staged + unstaged separately)"; git diff --cached; git diff; }`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10 2>/dev/null || echo "(no commits yet)"`

## Your task

変更内容を論理的な単位に分割し、複数のコミットに分けて順次作成します。

1. **変更の分析**: diffとgit statusから論理的に独立した変更単位を特定
   - 機能追加・バグ修正・リファクタ・ドキュメント・設定変更など目的別に分離
   - 無関係な変更を1つのコミットにまとめない
   - 単一目的の変更しかない場合は分割せず1コミットとする
2. **分割案の提示**: AskUserQuestionで分割案をユーザーに提示し承認を受ける
   - 各コミットに含めるファイルのみを列挙（メッセージはこの段階では確定させない）
   - 分割が不要な場合は単一コミット案として提示
3. **順次コミット実行**: 承認された分割案に従って先頭から順にコミット
   - `git reset` で一度ステージをクリアしてから、対象ファイルのみを `git add`
   - **コミット直前にAskUserQuestionで3種類のコミットメッセージ案を提示**し、ユーザーが選択した案でコミット
     - 3案はそれぞれ異なる観点を持たせる（例: 機能観点 / 影響観点 / 簡潔版、プレフィックスのバリエーション、抽象度の違いなど）
     - すべて制約（日本語・Conventional Commits形式・1行）を満たすこと
   - すべての分割が完了するまで繰り返す

## Constraint

- Claude co-authorshipフッターは不要
- メッセージは日本語（プレフィックスのみ英語、Conventional Commits形式）
- 1行で完結（本文なし）
- 分割単位はファイル単位（同一ファイル内の hunk 分割は行わない）
- 同一ファイルに複数の目的が混在している場合はその旨をユーザーに伝え、まとめて1コミットとするか確認する
