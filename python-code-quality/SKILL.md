---
name: python-code-quality
description: コーディングテスト提出用Pythonコードの品質規約。可読性・保守性・一貫性が採点対象となるため、Pythonコードを書く・レビューする・リファクタリングする際は必ずこのスキルを参照すること。「コードを綺麗にして」「可読性を上げて」「レビューして」という依頼でも必ず使うこと。
---

# Python コード品質規約（コーディングテスト提出用）

採点基準は「クリーンなコードか」「可読性があるか」「チーム開発を意識した保守性」。以下の規約を全ての提出コードに適用する。

## 命名

- **意味のある名前**: `data`, `tmp`, `result2` のような無情報な名前を避ける。仕様上の用語をそのまま使う
- 関数は動詞句: `calculate_total_price()`, `parse_order_line()`
- bool は疑問形: `is_valid`, `has_discount`, `can_ship`
- 定数は大文字スネークケース + モジュール冒頭: `MAX_RETRY_COUNT = 3`
- 慣習的な短縮名（ループの `i, j`、行列の `n, m`）は許容。それ以外の省略は禁止

## 関数設計

- 1関数1責務。目安は20〜30行以内。超えたら分割を検討
- 引数は4個以内を目安に。超えるなら dataclass にまとめる
- 早期リターンでネストを浅くする（ガード節）
- 副作用（グローバル変更、I/O）とロジックを分離する。ロジック関数は純粋関数に近づける

## マジックナンバーの排除

```python
# NG
if age >= 65:
    price *= 0.8

# OK
SENIOR_AGE_THRESHOLD = 65
SENIOR_DISCOUNT_RATE = 0.8
if age >= SENIOR_AGE_THRESHOLD:
    price *= SENIOR_DISCOUNT_RATE
```

## 型ヒントと docstring

- 全ての関数に型ヒントを付ける（Python 3.9+ の組み込みジェネリクス: `list[int]`, `dict[str, int]`）
- 公開関数には docstring。**What ではなく Why を書く**:

```python
def normalize_scores(scores: list[float]) -> list[float]:
    """スコアを 0-1 に正規化する。

    仕様3.2項の「相対評価」を実現するため min-max 正規化を採用。
    全要素が同値の場合は仕様に従い全て 0.5 を返す。
    """
```

- 自明なコメントは書かない（`i += 1  # iに1を足す` は禁止）
- 書くべきコメント: 解法の方針、計算量、仕様の曖昧箇所への解釈、非自明なトリック

## Pythonic なイディオム

- 内包表記（ただしネスト2段まで。複雑なら for に戻す）
- `enumerate`, `zip`, アンパックを活用。`range(len(x))` を避ける
- `with` によるリソース管理
- 文字列は f-string
- 比較の連鎖: `if 0 <= x < n`
- `collections.Counter`, `defaultdict` などの標準ライブラリを車輪の再発明せず使う

## エラー処理

- 例外を握りつぶさない（`except: pass` 禁止）
- 捕捉する例外型を明示: `except ValueError:`
- 仕様にエラー時の挙動が定義されていればそれに厳密に従う。未定義なら解釈をコメントに残す

## 提出前の機械チェック

環境にあれば以下を実行する:

```bash
python -m py_compile solution.py   # 構文チェック
ruff check solution.py             # リント（あれば）
ruff format solution.py            # フォーマット（あれば）
```

ruff がなければ PEP 8 準拠を目視確認する（インデント4、行長は目安88〜100、import順: 標準→サードパーティ→ローカル）。
