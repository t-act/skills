# 頻出アルゴリズムパターン選定ガイド

問題の特徴からパターンを選定するためのリファレンス。

## 問題の特徴 → 手法のマッピング

| 問題の特徴 | 候補手法 |
|---|---|
| 「最大値/最小値を求めよ」+ 単調性がある | 答えで二分探索 |
| 区間の和を何度も問われる | 累積和、BIT（Fenwick Tree） |
| 連続部分列で条件を満たす最長/個数 | しゃくとり法（two pointers） |
| 直近/最小/最大を動的に取り出す | heapq |
| 「〜通りの方法」「最適な選び方」+ 部分構造 | DP |
| グリッド/迷路の最短距離 | BFS（重みなし）、Dijkstra（重みあり） |
| 依存関係、順序制約 | トポロジカルソート |
| グループ分け、連結判定 | Union-Find |
| 文字列の一致・検索 | in演算子、辞書、（高度なら）Z-algorithm |
| 括弧の対応、直近の要素との比較 | スタック（単調スタック含む） |
| 組み合わせの数え上げ + mod | 階乗前計算 + 逆元 |
| ゲーム、ターン制 | Grundy数、後退解析 |

## Python標準ライブラリの活用

```python
from collections import deque, defaultdict, Counter
from heapq import heappush, heappop
from bisect import bisect_left, bisect_right, insort
from itertools import permutations, combinations, product, accumulate
from functools import lru_cache
import math  # gcd, lcm, comb, isqrt
```

- `Counter` : 頻度集計。`most_common()` が便利
- `accumulate` : 累積和を1行で
- `bisect` : ソート済みリストへの二分探索
- `math.comb(n, k)` : 二項係数（mod不要な場合）
- `lru_cache` : メモ化再帰（ただし深い再帰は反復DPに書き換える）

## 実装スニペット

### Union-Find

```python
class UnionFind:
    def __init__(self, n: int) -> None:
        self.parent = list(range(n))
        self.size = [1] * n

    def find(self, x: int) -> int:
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x

    def union(self, x: int, y: int) -> bool:
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        if self.size[rx] < self.size[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        self.size[rx] += self.size[ry]
        return True
```

### Dijkstra

```python
from heapq import heappush, heappop

def dijkstra(graph: list[list[tuple[int, int]]], start: int) -> list[int]:
    """graph[u] = [(v, weight), ...]"""
    inf = float("inf")
    dist = [inf] * len(graph)
    dist[start] = 0
    heap = [(0, start)]
    while heap:
        d, u = heappop(heap)
        if d > dist[u]:
            continue
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heappush(heap, (dist[v], v))
    return dist
```

### 答えで二分探索

```python
def binary_search_answer(lo: int, hi: int, is_ok) -> int:
    """is_ok(x) が単調（False...False True...True）のとき最小の True 位置を返す"""
    while lo < hi:
        mid = (lo + hi) // 2
        if is_ok(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

### BFS（グリッド）

```python
from collections import deque

def bfs_grid(grid: list[str], start: tuple[int, int]) -> list[list[int]]:
    h, w = len(grid), len(grid[0])
    dist = [[-1] * w for _ in range(h)]
    sr, sc = start
    dist[sr][sc] = 0
    queue = deque([start])
    while queue:
        r, c = queue.popleft()
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < h and 0 <= nc < w and grid[nr][nc] != "#" and dist[nr][nc] == -1:
                dist[nr][nc] = dist[r][c] + 1
                queue.append((nr, nc))
    return dist
```
