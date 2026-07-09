---
name: html-style-guide
description: HTML を生成・出力するときに参照する統一スタイルガイド。Anthropic Claude 風(温かいクリーム地 × セリフ見出し × コーラルの単一アクセント)のデザイントークン・タイポグラフィ・レイアウト・コンポーネント(表・コード・コード差分・コールアウト)をライト/ダーク両対応で定義する。論文要約・記事・解説などの読み物、データ/比較表/レポート、コードレビュー資料(コード差分表示)を単一 HTML ファイルで作るときに必ずこのガイドに従う。
user-invocable: true
allowed-tools: Read, Write, AskUserQuestion
argument-hint: "(参照用スタイルガイド。HTML を出力するスキル・作業から参照する)"
---

# HTML 統一スタイルガイド

HTML を出力するあらゆる場面で参照する、スタイル統一のための共有ガイド。
これ自体は単発でも呼べるが、主な役割は **他のスキルや作業から参照される「デザインの正本」** であること。HTML を書き出す前にこのガイドのトークンとコンポーネント指針に従う。

基調は Anthropic Claude.com の世界観 —— **温かいクリーム地 × セリフ見出し × コーラルの単一アクセント** —— を踏襲する。雑誌のロングフォーム記事のように余白を広く取り、セリフ見出しと humanist sans 本文を組み合わせる。`paper-summary` スキルと同じ設計言語を共有するが、こちらは読み物に加えて **データ/レポート** と **コードレビュー(コード差分)** まで扱い、**ライト/ダーク両対応** を持つ。

## 0. 位置づけと使い方

- **参照する側**: HTML を出力するとき、まずこのファイルの §2 トークン・§3 タイポ・§4 レイアウトを `<head>` の `<style>` に取り込み、必要なコンポーネント(§5)だけを足す。
- **単発利用**: 入力(md/テキスト/差分など)を渡されたら、その内容に合ったコンポーネントを選んでこのスタイルで単一 HTML を書き出す。
- **`paper-summary` は独立**: 既存の `paper-summary` は自身にスタイル定義を内蔵しており、このガイドとは独立に運用する。こちらを `paper-summary` に強制適用しない。

## 1. 共通の大原則

- **単一 HTML ファイル**。外部 CSS / JS / フォント / 画像に依存しない。CSS は `<head>` 内の `<style>` に内蔵し、画像が要るなら data URI で埋める。
- **`<meta name="viewport" content="width=device-width, initial-scale=1">` を必ず入れる**。レスポンシブ。
- **本文はウィンドウ幅に追従**させつつ、可読性のため **`max-width` で頭打ち**にして中央寄せ(読み物 `820px` / データ・レポート `960px` / コードレビュー `1100px` を目安)。本文面の左右には常に余白を残す。
- **影は控えめ (flat)**。区切りはヘアライン (`--hairline`) か面色の差(クリーム ↔ カード ↔ 濃色)で表現する。強い装飾的な影・グラデーションは使わない。
- **アクセントは 1 色**。`--primary`(Coral)をリンク・強調・コールアウトに使う。teal/amber は補助でごく稀に。
- **色・寸法はハードコードせず CSS 変数を参照**する。hex を本文 CSS に直接散らさない。
- **ライト/ダーク両対応**。§2 の方式で `prefers-color-scheme` に追従し、`data-theme` 明示切替も許容する。

## 2. デザイントークン(CSS 変数・ライト/ダーク)

`<head>` の `<style>` 冒頭に以下を置く。ライトを既定にし、`@media (prefers-color-scheme: dark)` でダークへ上書き、さらに `:root[data-theme="..."]` の明示指定を最優先にする(将来トグルを付けても効くように)。

```css
:root {
  /* ---- color: light (既定) ---- */
  --primary: #cc785c;          /* Coral — 唯一のアクセント(リンク・CTA・強調) */
  --primary-active: #a9583e;   /* コーラルの濃いめ(アクティブ・削除記号) */
  --ink: #141413;              /* 見出し・強調テキスト(温かみのある黒) */
  --body: #3d3d3a;             /* 本文 */
  --body-strong: #252523;      /* 強調段落・リード */
  --muted: #6c6a64;            /* 小見出し・キャプション */
  --muted-soft: #8e8b82;       /* 注記・細字・行番号 */
  --canvas: #faf9f5;           /* ページの地(温かいクリーム。純白にしない) */
  --surface-soft: #f5f0e8;     /* 区切り帯・hunk 見出し・淡いバンド */
  --surface-card: #efe9de;     /* カード・TL;DR・表ヘッダ(canvas より一段濃い) */
  --surface-dark: #181715;     /* 濃色面(強調引用) */
  --code-bg: #1f1e1b;          /* コードブロックの地 */
  --on-dark: #faf9f5;          /* 濃色面上の文字(クリーム白) */
  --on-dark-soft: #a09d96;     /* 濃色面上の弱い文字 */
  --on-primary: #ffffff;       /* コーラル面上の文字 */
  --hairline: #e6dfd8;         /* クリーム面の 1px 罫 */
  --hairline-soft: #ebe6df;    /* 同一バンド内のごく淡い区切り */
  --accent-teal: #5db8a6;      /* 補助アクセント(ごく稀に) */
  /* ---- code diff: light ---- */
  --diff-add-bg: #e9f1e2;      /* 追加行の地(淡い緑クリーム) */
  --diff-add-ink: #3f6b34;     /* 追加行のガター記号 "+" */
  --diff-add-bar: #7aa860;     /* 追加行の左アクセントバー */
  --diff-del-bg: #f7e7df;      /* 削除行の地(淡いコーラル寄り) */
  --diff-del-ink: #a9583e;     /* 削除行のガター記号 "-" */
  --diff-del-bar: #cc785c;     /* 削除行の左アクセントバー */
  --diff-hunk-bg: #f5f0e8;     /* @@ hunk 見出しの地 */
  --diff-hunk-ink: #6c6a64;    /* @@ hunk 見出しの文字 */
  --diff-gutter: #8e8b82;      /* 行番号ガター */
  --diff-word-add: #cfe6bf;    /* 語単位ハイライト(追加) */
  --diff-word-del: #f4cdbc;    /* 語単位ハイライト(削除) */
  /* ---- radius ---- */
  --r-sm: 6px; --r-md: 8px; --r-lg: 12px; --r-xl: 16px; --r-pill: 9999px;
  /* ---- spacing (4px ベース) ---- */
  --s-xs: 8px; --s-sm: 12px; --s-md: 16px; --s-lg: 24px; --s-xl: 32px; --s-xxl: 48px; --s-section: 96px;
}

/* ダーク: OS 設定に追従 */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --primary: #d88c6e;
    --primary-active: #e0a488;
    --ink: #f5f2ea;
    --body: #cfccc2;
    --body-strong: #e6e2d8;
    --muted: #9a968c;
    --muted-soft: #78756c;
    --canvas: #1a1917;         /* 温かい黒(クリームの反転。真っ黒にしない) */
    --surface-soft: #221f1b;
    --surface-card: #27241f;
    --surface-dark: #0f0e0c;
    --code-bg: #211f1b;
    --on-dark: #f5f2ea;
    --on-dark-soft: #9a968c;
    --on-primary: #1a1917;
    --hairline: #35322c;
    --hairline-soft: #2c2925;
    --accent-teal: #6ec2b1;
    --diff-add-bg: #1e2a1b;
    --diff-add-ink: #86c06a;
    --diff-add-bar: #4f7c3a;
    --diff-del-bg: #2e1e1a;
    --diff-del-ink: #dc9077;
    --diff-del-bar: #a9583e;
    --diff-hunk-bg: #221f1b;
    --diff-hunk-ink: #9a968c;
    --diff-gutter: #6a675f;
    --diff-word-add: #2f4a26;
    --diff-word-del: #4a2c22;
  }
}

/* ダーク: data-theme で明示指定(トグル用・OS 設定より優先) */
:root[data-theme="dark"] {
  /* 上記ダークと同じ値をここにも展開する(明示切替を OS 設定に勝たせるため) */
}
```

> 実装メモ: ダークの値は「@media 版」と「`[data-theme="dark"]` 版」で同一にする。重複を避けたい場合は、ダーク値を `:root[data-theme="dark"], :root:not([data-theme="light"]) { ... }` を `@media` 内に入れる形でまとめてよい。トグル UI が不要なら `@media (prefers-color-scheme: dark)` だけでも成立する。

## 3. タイポグラフィ

Copernicus / StyreneB は Anthropic 専用フォントのため、オープンな代替を使う。日本語は見出しに明朝、本文にゴシックを当ててセリフ/サンセリフの対比を保つ。

- **見出し(セリフ)**: `"Tiempos Headline", "Cormorant Garamond", "EB Garamond", "Hiragino Mincho ProN", "Yu Mincho", Garamond, serif`。**weight 400(太字にしない)** + 負の letter-spacing。
  - h1 約 44–48px / line-height 1.1 / letter-spacing -1px
  - h2 約 34–36px / -0.5px
  - h3 約 26–28px / -0.3px
  - 負のトラッキングを必ず効かせる(セリフの締まった「考え抜かれた」声がこのデザインの核)。
- **本文(humanist sans)**: `"Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", "Hiragino Sans", "Yu Gothic", sans-serif`。本文 16px / weight 400 / line-height 1.55。ラベル・強調句は weight 500。
- **コード/等幅**: `"JetBrains Mono", ui-monospace, SFMono-Regular, monospace`。14px / line-height 1.6。
- weight は 400 / 500 が基本。見出しは 400 のまま(Bold にしない)。geometric sans(Helvetica/Arial)は使わない(humanist の温かみが失われる)。

## 4. レイアウト

- ページ背景・本文面ともに `--canvas`。Apple 風の白カードで本文を囲わず、地に直接組むエディトリアル構成。
- 余白は 4px ベース(`--s-*`)。セクション間は広く(48–96px)、見出し上に十分な余白。カード内側は `--s-xl`(32px)程度とゆったり。
- リンクは `--primary`。下線はホバー時か、常時なら控えめに。
- 区切りは `--hairline` の 1px か面色差。縦罫は基本使わない。

## 5. コンポーネント別の当て方

### ヘッダー
タイトルを h1(セリフ)。副題・著者・日付・出典・外部リンクは `--muted` のキャプション。下端に `--hairline` の 1px 区切り。リンクは `--primary`。

### TL;DR / リード
`--surface-card` 面 + `--r-lg`(12px)角丸 + 左に `--primary` の 3–4px アクセントバー。3〜5 行の要点。

### 本文セクション
内容に応じて柔軟に(読み物なら「背景・手法・結果・議論」等)。本文色は `--body`、リードや強調段落は `--body-strong`。

### 表(データ・比較・レポート)
`<table>`。横罫は `--hairline` の 1px のみ、**縦罫なし**。ヘッダ行は `--surface-card` 背景 + weight 500。角丸 `--r-md`。数値は右寄せ・等幅を検討。強調したい行は `--surface-soft` で淡く帯掛け。widthが溢れる表は `overflow-x: auto` のラッパで包み、ページ本体が横スクロールしないようにする。

### コールアウト / 強調
- 通常の補足: `--surface-card` 面 + 左アクセントバー(`--muted` か `--primary`)。
- 特に強調したい結論: coral 面(`--primary` 背景 + `--on-primary` 文字 + `--r-lg`)を稀に。
- 強調引用や 1 節のコード塊: 濃色面(`--surface-dark` 背景 + `--on-dark` 文字)を使ってよい。多用しない。

### コード(インライン / ブロック)
- インライン: `--surface-soft` 地 + `--r-sm` + `JetBrains Mono`、`--body-strong` 文字。
- ブロック: `--code-bg` 地 + `--on-dark` 文字(ダーク時は面が少し明るくなる)+ `--r-lg`。`overflow-x: auto` で包む。行番号を付けるなら `--muted-soft`。

### コード差分(コードレビュー資料) ★重要
**基本は unified(1 カラム)**。追加=緑系、削除=赤(コーラル寄り)系、文脈行=地色。横幅の広いレビューで対応行を突き合わせたいときだけ **side-by-side(2 カラム)** を使う。どちらも同じトークンを使う。

共通ルール:
- コード部は `JetBrains Mono` / `white-space: pre` / `overflow-x: auto`(ラッパで包み、ページ本体は横スクロールさせない)。
- 各差分ブロックの先頭にファイルパス見出し(`--surface-soft` 地 + `--muted` + 等幅)を置く。
- `@@ ... @@` の hunk 見出しは `--diff-hunk-bg` / `--diff-hunk-ink`。
- ガター(記号列 `+`/`-`/空)と行番号は選択・コピー時にコードへ混ざらないよう `<td>` を分ける(下記構造)。行番号・記号は `user-select: none`。
- 大きな変更内では語単位ハイライト(`--diff-word-add` / `--diff-word-del`)を任意で使う。

**unified の構造(既定)** —— 旧行番号・新行番号・記号・コードの 4 列テーブル:

```html
<figure class="diff">
  <figcaption class="diff-file">src/app/main.py</figcaption>
  <table class="diff-table">
    <tbody>
      <tr class="diff-hunk">
        <td class="ln" colspan="3"></td><td class="code">@@ -10,6 +10,7 @@ def run():</td>
      </tr>
      <tr class="diff-ctx">
        <td class="ln old">10</td><td class="ln new">10</td><td class="sign"></td><td class="code">    total = 0</td>
      </tr>
      <tr class="diff-del">
        <td class="ln old">11</td><td class="ln new"></td><td class="sign">-</td><td class="code">    return total</td>
      </tr>
      <tr class="diff-add">
        <td class="ln old"></td><td class="ln new">11</td><td class="sign">+</td><td class="code">    total += 1</td>
      </tr>
      <tr class="diff-add">
        <td class="ln old"></td><td class="ln new">12</td><td class="sign">+</td><td class="code">    return total</td>
      </tr>
    </tbody>
  </table>
</figure>
```

対応 CSS(抜粋):

```css
.diff { margin: var(--s-lg) 0; border: 1px solid var(--hairline); border-radius: var(--r-lg); overflow: hidden; }
.diff-file { padding: var(--s-xs) var(--s-md); background: var(--surface-soft); color: var(--muted);
  font-family: "JetBrains Mono", ui-monospace, monospace; font-size: 13px; border-bottom: 1px solid var(--hairline); }
.diff-table { width: 100%; border-collapse: collapse; font-family: "JetBrains Mono", ui-monospace, monospace;
  font-size: 13.5px; line-height: 1.55; }
.diff-table td { padding: 1px 0; vertical-align: top; }
.diff-table .ln { width: 1%; padding: 1px 10px; text-align: right; color: var(--diff-gutter);
  user-select: none; white-space: nowrap; }
.diff-table .sign { width: 1ch; padding: 1px 6px; text-align: center; user-select: none; }
.diff-table .code { padding: 1px 12px 1px 4px; white-space: pre; overflow-x: auto; color: var(--body-strong); }
.diff-hunk td { background: var(--diff-hunk-bg); color: var(--diff-hunk-ink); padding: 4px 12px; }
.diff-add { background: var(--diff-add-bg); box-shadow: inset 3px 0 0 var(--diff-add-bar); }
.diff-add .sign { color: var(--diff-add-ink); }
.diff-del { background: var(--diff-del-bg); box-shadow: inset 3px 0 0 var(--diff-del-bar); }
.diff-del .sign { color: var(--diff-del-ink); }
.diff-word-add { background: var(--diff-word-add); border-radius: 3px; }
.diff-word-del { background: var(--diff-word-del); border-radius: 3px; }
```

**side-by-side(横幅が長いときのみ)** —— 旧番号・旧コード・新番号・新コードの 4 列。削除だけの行は右を空セル、追加だけの行は左を空セルにする:

```html
<figure class="diff diff-split">
  <figcaption class="diff-file">src/app/main.py</figcaption>
  <table class="diff-table">
    <tbody>
      <tr>
        <td class="ln old">11</td><td class="code side-del">    return total</td>
        <td class="ln new">11</td><td class="code side-add">    total += 1</td>
      </tr>
      <tr>
        <td class="ln old"></td><td class="code side-empty"></td>
        <td class="ln new">12</td><td class="code side-add">    return total</td>
      </tr>
    </tbody>
  </table>
</figure>
```

```css
.diff-split .side-del { background: var(--diff-del-bg); box-shadow: inset 3px 0 0 var(--diff-del-bar); }
.diff-split .side-add { background: var(--diff-add-bg); box-shadow: inset 3px 0 0 var(--diff-add-bar); }
.diff-split .side-empty { background: var(--surface-soft); }
```

side-by-side は狭い画面で窮屈になるため、`@media (max-width: 720px)` では unified に相当する縦積みへフォールバックするか、`overflow-x: auto` で横スクロールを許す。デフォルトの選択に迷ったら unified を使う。

### 個人的ポイント / まとめ
末尾に置き、`--surface-card` 面で本文と差をつける。

## 6. 他スキル・作業からの参照

- HTML を生成するスキル(要約・レポート・レビュー資料など)は、テンプレートを固定せず **このガイドのトークンと該当コンポーネントだけ** を `<style>` に取り込む。
- 読み物なら §5 のヘッダー/TL;DR/表/コールアウト、レポートなら表中心、コードレビューなら §5 のコード差分を主役にする。
- 出力先やファイル名は参照元スキルの規約に従う(このガイドは配色と構造のみを規定する)。

## 7. 出力前チェックリスト

- [ ] 単一 HTML・外部依存なし・`<style>` は `<head>` 内
- [ ] `viewport` メタタグあり / ページ本体が横スクロールしない(広い表・コードは `overflow-x: auto` ラッパ)
- [ ] 色・寸法は CSS 変数参照(hex 直書きを本文 CSS に散らしていない)
- [ ] 見出しはセリフ weight 400 + 負トラッキング / 本文は humanist sans
- [ ] 影は控えめ(flat)・区切りはヘアラインか面色差
- [ ] アクセントは `--primary` 1 色中心
- [ ] ライト/ダーク両方で可読(`prefers-color-scheme` 追従)
- [ ] コード差分は基本 unified・記号/行番号は `user-select: none`・side-by-side は横長時のみ
