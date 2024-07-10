---
title: AtCoder Social Distance on Graph
date: 2024-07-07 00:00:00 +0800
categories: []
tags: [atcoder] # TAG names should always be lowercase
comments: false
math: true
---

[Back](https://pyjuan91.github.io/posts/atcoder-plan/)

[題目連結](https://atcoder.jp/contests/arc165/tasks/arc165_c){:target="\_blank"}

## 題意

給定一個 simple connected undirected weighted graph，\
有 $$N <= 2 * 10^5$$ 個節點，$$M <= 2 * 10^5$$ 條權邊。\
根據你的喜好，將每個節點著上紅色或藍色，有\
$$X$$ 為任意兩**相同顏色**節點距離的最小值。

在某種著色方法下，$$X$$ 會有最大值，求此時的最大值為何？

---

## 分析

官方的題解為使用**二分搜尋**猜測 $$X$$ 的值後，用**二分圖著色**求解。（但是我不會）

根據 World Final 助教的建議，**樹問題先想若是鏈能否解決，圖問題先想若是樹能否解決**。\
若這題改為樹題，則直接交錯著色變為最佳解。

> Obviously, 兩條路徑和必大於單條路徑和
{: .prompt-info}

若今天考慮圖題，則環是處理關鍵，\
觀察三節點環，知道將**最長邊**著同色會有最佳解，
奇點環與三點環相同，偶點環交錯著色最佳（與樹同）。

因此建立**最小生成樹**便有圖上的最佳解，最後檢查非樹邊是否有比樹上更小的權即可。

---

## 實作

製作最小生成樹，並標記圖邊是否為樹邊

```c++
  sort(edges.begin(), edges.end());
  atcoder::dsu d(n);
  vector<set<pair<int, int>>> tree(n);
  vector<bool> is_tree_edge;
  for (auto [w, u, v] : edges) {
    if (d.same(u, v)) {
      is_tree_edge.push_back(false);
      continue;
    }
    d.merge(u, v);
    tree[u].emplace(w, v);
    tree[v].emplace(w, u);
    is_tree_edge.push_back(true);
  }
```

計算樹上的 $$X$$ 值

```c++
  int64_t ans = INT64_MAX;
  for (int i = 0; i < n; i++) {
    if (tree[i].size() > 1) {
      auto [w1, v1] = *tree[i].begin();
      auto [w2, v2] = *(++tree[i].begin());
      ans = min(ans, (int64_t)w1 + w2);
    }
  }
```

將樹著色後檢查圖邊的 $$X$$ 值

```c++
  vector<int> color(n, -1);
  auto dfs = [&](auto self, int u, int c) -> void {
    color[u] = c;
    for (auto [w, v] : tree[u]) {
      if (color[v] == -1) {
        self(self, v, c ^ 1);
      }
    }
    };
  dfs(dfs, 0, 0);

  for (int i = 0; i < m; i++) {
    if (is_tree_edge[i]) {
      continue;
    }
    auto [w, u, v] = edges[i];
    if (color[u] == color[v]) {
      ans = min(ans, (int64_t)w);
    }
  }
```

[Submission](https://atcoder.jp/contests/arc165/submissions/55422814){:target="\_blank"}


[Back](https://pyjuan91.github.io/posts/atcoder-plan/)