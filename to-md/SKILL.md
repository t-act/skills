---
name: to-md
description: markitdown を使って PDF / Word / Excel / PowerPoint などのファイルを Markdown に変換する
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(uvx:*), Bash(ls:*), Bash(test:*), Bash(mkdir:*), Bash(dirname:*), Bash(basename:*), AskUserQuestion
argument-hint: "<入力ファイル> [出力先パス]"
---

# ファイル → Markdown 変換: $ARGUMENTS

`markitdown` (Microsoft 製) を `uvx` 経由で実行し、任意の入力ファイルを Markdown に変換する。

## 1. 引数の解析

`$ARGUMENTS` から以下を抽出する:

- **入力ファイル** (必須): 第1引数。未指定ならエラーとして報告し終了
- **出力先** (任意): 第2引数
  - 未指定: `<入力ファイルのディレクトリ>/<basename>.md`
  - ディレクトリ (末尾が `/` または既存ディレクトリ) を指定: `<指定ディレクトリ>/<basename>.md`
  - ファイル名を指定: そのパスをそのまま使う

`basename` は入力ファイルから拡張子を除いた部分。例: `/path/to/hoge.pdf` → `hoge`。

## 2. 入力ファイルの検証

- `test -f "<入力ファイル>"` で存在確認
- 存在しなければ「入力ファイルが見つかりません: <path>」と報告して終了

## 3. 出力パスの確定と上書き確認

- 出力先ディレクトリが無ければ `mkdir -p` で作成
- `test -f "<出力先>"` で既存チェック。存在する場合、`AskUserQuestion` で「上書きする / キャンセル」を確認
  - キャンセルなら終了

## 4. 変換の実行

```bash
uvx --from "markitdown[all]" markitdown "<入力ファイル>" -o "<出力先>"
```

- 初回実行時は依存パッケージのダウンロードが走る (数十 MB)
- 成功したら `変換完了: <入力> -> <出力>` を1行で報告
- 失敗したら stderr の要点を返す

## 5. 制約

- 出力先以外には絶対にファイルを書き込まない (`output/` 等のディレクトリを作らない)
- 既存ファイルの上書きは必ず事前確認を取る
- 対応フォーマット (PDF / DOCX / XLSX / PPTX / HTML / CSV / 画像 / 音声 等) は markitdown が自動判定するため、拡張子ごとの分岐は不要
