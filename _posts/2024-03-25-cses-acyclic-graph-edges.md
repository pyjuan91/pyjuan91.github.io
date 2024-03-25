---
title: CSES Acyclic Graph Edges
date: 2024-03-25 00:00:00 +0800
categories: []
tags: [cses]     # TAG names should always be lowercase
comments: false
math: true
---

[題目連結](https://cses.fi/problemset/task/1756){:target="_blank"}

## 題意

給定 $$n$$ 點 $$m$$邊 的**無向（可能有環）**圖。

替每個無向邊配置方向，使其變成有向無環圖。

---

第三周平面圖的作業之一，看起來可以用 $$DFS$$ 樹不可往回拉邊的方式求解。

## 不太聰明的建立$$DFS$$樹

這我，在寫很多圖論題也只會靠套 template 湊出答案，思考力低下 😭

1. 存圖時對邊多加一個 tag ，在 DFS 時決定方向。
2. 遇到新點直接確定方向（新點不會造成 cycle ）
3. 遇到舊點留高點往低點接的方向（深度小接向深度大）

> 如果往**同深度**或是**高點**回接必會造成 cycle ❗
>
> (Because there's **always** a path from high position to low position.)
{: .prompt-info}

``` c++
#include <bits/stdc++.h>
using namespace std;

const int kMax = 1e5 + 5;
int n, m, u, v, depth[kMax];
bool vis[kMax];
vector<pair<int, int>> g[kMax];

void dfs(int u, int p) {
  vis[u] = true;
  depth[u] = depth[p] + 1;
  for (auto& [v, w] : g[u]) {
    if (!vis[v])
      w = 1, dfs(v, u);
    else if (depth[v] > depth[u])
      w = 1;
  }
}

int32_t main() {
  cin.tie(nullptr)->sync_with_stdio(false);

  cin >> n >> m;
  while (m--) {
    cin >> u >> v;
    g[u].emplace_back(v, 0);
    g[v].emplace_back(u, 0);
  }

  for (int i = 1; i <= n; i++)
    if (!vis[i]) dfs(i, 0);

  for (int i = 1; i <= n; i++) {
    for (auto& [v, w] : g[i]) {
      if (w == 1) cout << i << ' ' << v << '\n';
    }
  }

  return 0;
}
```

### 時間複雜度

* IO 是$$O(m)$$，DFS 是$$O(n+m)$$ 算高效

## 頂級理解

這不是我，我是在繳交時看到 virtual judge 上居然有長度極短的程式碼才發現。

> 完全不需要建立 DFS Tree，只要確保每個 Node 都是小編號接大編號 (or vice versa) 
> 便一定不會有 cycle（太神了 OTn）
{: .prompt-tip}

* 扎實$$O(m)$$的時間複雜度

``` c++
#include <bits/stdc++.h>
using namespace std;
int main() {
  int n, m, u, v;
  cin >> n >> m;
  while (m--) cin >> u >> v, cout << min(u, v) << ' ' << max(u, v) << '\n';
}
```
