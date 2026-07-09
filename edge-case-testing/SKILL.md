---
name: edge-case-testing
description: コーディングテストの解答を提出前に検証するためのエッジケース生成・テスト実行ワークフロー。境界値の洗い出し、テストケース自動生成、愚直解との突き合わせ（ランダムテスト）、実行時間計測を行う。解答コードが書き上がったとき、「テストして」「検証して」「エッジケースを確認して」と言われたとき、提出前の最終確認時には必ずこのスキルを使うこと。
---

# エッジケーステスト ワークフロー

エッジケース対応は明示的な採点対象である。解答コードが完成したら、必ず以下を実行してから提出する。

## Step 1: エッジケースの洗い出し

問題の制約表と突き合わせ、該当するものを全てリスト化する:

**サイズ系**
- 最小入力（N=0, N=1, 空文字列, 空リスト）
- 最大入力（制約上限ちょうど）
- 要素数1と2（ループ・比較の境界）

**値系**
- 全要素が同値 / 全要素が異なる
- 最小値・最大値ちょうど（0, 1, 10^9, -10^9 など）
- 負数、ゼロの混在（仕様上あり得るなら）
- 重複要素

**構造系**
- ソート済み / 逆順 / ランダム
- グラフ: 非連結、自己ループ、多重辺、直線、星型、完全グラフ
- 文字列: 全て同一文字、回文、大文字小文字混在
- グリッド: 1行/1列のみ、全マス壁、スタート=ゴール

**仕様系**
- 「ただし」「〜の場合」で定義された例外仕様の発火条件ちょうど
- 同点・同順位のタイブレーク
- 丸め境界（0.5 など。Python の round は銀行丸めであることに注意）

## Step 2: テストハーネス作成

標準入出力形式の場合:

```bash
mkdir -p tests
# tests/case01.in / tests/case01.out にケースを配置
for f in tests/*.in; do
  base="${f%.in}"
  actual=$(python solution.py < "$f")
  expected=$(cat "$base.out")
  if [ "$actual" == "$expected" ]; then
    echo "PASS: $base"
  else
    echo "FAIL: $base"
    diff <(echo "$expected") <(echo "$actual")
  fi
done
```

関数形式・実装問題の場合は pytest でパラメタライズする:

```python
import pytest
from solution import solve

@pytest.mark.parametrize("args, expected", [
    ((1, [5]), 5),          # 最小入力
    ((3, [2, 2, 2]), 6),    # 全同値
    # 要件IDごとに1ケース以上
])
def test_solve(args, expected):
    assert solve(*args) == expected
```

## Step 3: ランダムテスト（愚直解との突き合わせ）

正解が自明でない場合、遅くても確実に正しい愚直解（brute force）を別途書き、ランダム入力で本解と比較する:

```python
import random

def brute_force(n, values):  # O(N^2)でも良いので確実に正しい実装
    ...

for trial in range(1000):
    n = random.randint(1, 8)          # 小さいサイズで網羅的に
    values = [random.randint(-10, 10) for _ in range(n)]
    expected = brute_force(n, values)
    actual = solve(n, values)
    assert actual == expected, f"MISMATCH: n={n}, values={values}, expected={expected}, actual={actual}"
print("all random tests passed")
```

不一致が出たら、その最小ケースを固定テストに追加してから修正する。

## Step 4: 性能検証

最大制約の入力を生成し、実行時間を計測する:

```bash
python generate_max_case.py > tests/max.in
time python solution.py < tests/max.in > /dev/null
```

制限時間の50%以内を合格ラインとする。超える場合は algo-problem スキルの `references/python-perf.md` を参照。

## Step 5: 結果報告

ユーザーに以下を報告する:

- 実行したテストの分類と件数（サンプル / 境界値 / ランダム / 性能）
- 全件の PASS/FAIL
- 発見・修正したバグの内容
- 残存リスク（テストで担保できていない仕様があれば明示）
