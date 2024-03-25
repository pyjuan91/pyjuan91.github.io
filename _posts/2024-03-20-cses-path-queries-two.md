---
title: CSES Path Queries II (更新的坑人測資)
date: 2024-03-20 00:00:00 +0800
categories: []
tags: [cses]     # TAG names should always be lowercase
comments: false
math: true
---

[題目連結](https://cses.fi/problemset/task/2134/){:target="_blank"}

## 題意

給一個 $$n$$ 個節點的樹，並需要操作 $$q$$ 個指令 ($$n$$, $$q$$ 都是 $$2e5$$ 等級)
指令有兩種：
1. 單點更新樹的值
2. 找出 $$u$$ 到 $$v$$ 的路徑上，所有點中的最大值

---

這題是`heavy-light decomposition`的模板(?題 (中文叫樹鏈剖分)

雖然是模板題，但是第九筆的測資非常坑人，用最 general 的 template 會 `TLE`

> 第九筆測資應該是judge新增的，因為網路上的一些舊題解都會在第九筆測資TLE (e.g. [USACO Guide 的題解](https://usaco.guide/plat/hld?lang=cpp){:target="_blank"}  ，不過已向他們發issue 應該很快會更新)
{: .prompt-info}

## USACO Guide Solution

先看美國人的解～雖然這個解會超時，但是方法是正確的。
1. 將樹拆成多條**重鍊** （使用兩次 $$dfs$$）
2. 使用單點更新的最大值線段樹維護這些**重鍊**

> 就是 **Heavy-Light Decomposition** 的基礎用法，因為網路上教學很多，細節略～
{: .prompt-tip}

``` c++
#include "bits/stdc++.h"
using namespace std;

const int N = 2e5 + 5;
const int D = 19;
const int S = (1 << D);

int n, q, v[N];
vector<int> adj[N];

int sz[N], p[N][D], dep[N];
int st[S], id[N], tp[N];

void update(int idx, int val, int i = 1, int l = 1, int r = n) {
  if (l == r) {
    st[i] = val;
    return;
  }
  int m = (l + r) / 2;
  if (idx <= m)
    update(idx, val, i * 2, l, m);
  else
    update(idx, val, i * 2 + 1, m + 1, r);
  st[i] = max(st[i * 2], st[i * 2 + 1]);
}
int query(int lo, int hi, int i = 1, int l = 1, int r = n) {
  if (lo > r || hi < l) return 0;
  if (lo <= l && r <= hi) return st[i];
  int m = (l + r) / 2;
  return max(query(lo, hi, i * 2, l, m), query(lo, hi, i * 2 + 1, m + 1, r));
}

int dfs_sz(int cur, int par) {
  sz[cur] = 1;
  for (int chi : adj[cur]) {
    if (chi == par) continue;
    dep[chi] = dep[cur] + 1;
    p[chi][0] = cur;
    sz[cur] += dfs_sz(chi, cur);
  }
  return sz[cur];
}
void init_lca() {
  for (int d = 1; d < 18; d++)
    for (int i = 1; i <= n; i++) p[i][d] = p[p[i][d - 1]][d - 1];
}
int ct = 1;
void dfs_hld(int cur, int par, int top) {
  id[cur] = ct++;
  tp[cur] = top;
  update(id[cur], v[cur]);
  int h_chi = -1, h_sz = -1;
  for (int chi : adj[cur]) {
    if (chi == par) continue;
    if (sz[chi] > h_sz) {
      h_sz = sz[chi];
      h_chi = chi;
    }
  }
  if (h_chi == -1) return;
  dfs_hld(h_chi, cur, top);
  for (int chi : adj[cur]) {
    if (chi == par || chi == h_chi) continue;
    dfs_hld(chi, cur, chi);
  }
}
int lca(int a, int b) {
  if (dep[a] < dep[b]) swap(a, b);
  for (int d = D - 1; d >= 0; d--) {
    if (dep[a] - (1 << d) >= dep[b]) {
      a = p[a][d];
    }
  }
  for (int d = D - 1; d >= 0; d--) {
    if (p[a][d] != p[b][d]) {
      a = p[a][d];
      b = p[b][d];
    }
  }
  if (a != b) {
    a = p[a][0];
    b = p[b][0];
  }
  return a;
}
int path(int chi, int par) {
  int ret = 0;
  while (chi != par) {
    if (tp[chi] == chi) {
      ret = max(ret, v[chi]);
      chi = p[chi][0];
    } else if (dep[tp[chi]] > dep[par]) {
      ret = max(ret, query(id[tp[chi]], id[chi]));
      chi = p[tp[chi]][0];
    } else {
      ret = max(ret, query(id[par] + 1, id[chi]));
      break;
    }
  }
  return ret;
}
int main() {
  scanf("%d%d", &n, &q);
  for (int i = 1; i <= n; i++) scanf("%d", &v[i]);
  for (int i = 2; i <= n; i++) {
    int a, b;
    scanf("%d%d", &a, &b);
    adj[a].push_back(b);
    adj[b].push_back(a);
  }
  dfs_sz(1, 1);
  init_lca();
  memset(st, 0, sizeof st);
  dfs_hld(1, 1, 1);
  while (q--) {
    int t;
    scanf("%d", &t);
    if (t == 1) {
      int s, x;
      scanf("%d%d", &s, &x);
      v[s] = x;
      update(id[s], v[s]);
    } else {
      int a, b;
      scanf("%d%d", &a, &b);
      int c = lca(a, b);
      int res = max(max(path(a, c), path(b, c)), v[c]);
      printf("%d ", res);
    }
  }
}
```

### 時間複雜度

* IO 和兩次 dfs 是 $$O(n)$$ 
* 單點更新是 $$log(n)$$
* 詢問路徑最大值是 $$log(log(n))$$ 因為最多跳躍 $$log(n)$$ 條鍊，就可以抵達同一條，每次跳鍊會花 $$log(n)$$ 做線段樹詢問

Total is $$O(n + qloglogn)$$. 是可以通過的時間複雜度

但芬蘭人 (judge maintainer) 不會讓人這麼好過的 💖

![Desktop View](/assets/img/9tle.png){: w="500" }

---

## Revised AC Solution

這題是 2024 spring 日月卦長的競技程式設計二 其中一週（甚至被標示為**Easy**）的題目

試了大約三小時（通識課都沒在聽😵）終於靠壓常數過去了（時間複雜度一樣）

> 感謝網路上各種天才的 tricks，真的有夠香，參考Codeforces上一篇[神文](https://codeforces.com/blog/entry/18051){:target="_blank"}
{: .prompt-info}

### 堅果殼

用一個**低空間複雜度**且**高效率**的方法製作線段樹就可以AC了（方法參見Codeforces神文）

其餘的地方都相同。注意線段樹的建立不用特別拉出來做一次，在做重鍊剖分時同時做單點更新即可。

``` c++
#include <bits/stdc++.h>
#define int long long
using namespace std;
 
const int kMax = 2e5 + 5;
int n, m, op, x, y, z, cnt = 0;
int a[kMax] = {}, dep[kMax] = {}, fa[kMax] = {}, siz[kMax] = {},
    hson[kMax] = {};
int nid[kMax] = {}, ltop[kMax] = {};
int t[kMax << 1] = {};
vector<int> g[kMax];
 
void modify(int p, int value) {  // set value at position p
  for (t[p += n] = value; p > 1; p >>= 1) t[p >> 1] = max(t[p], t[p ^ 1]);
}
 
int query(int l, int r) {  // maximum on interval [l, r)
  int res = 0;
  for (l += n, r += n; l < r; l >>= 1, r >>= 1) {
    if (l & 1) res = max(res, t[l++]);
    if (r & 1) res = max(res, t[--r]);
  }
  return res;
}
 
void dfs1(int u, int f) {
  dep[u] = dep[f] + 1;
  fa[u] = f;
  siz[u] = 1;
  for (auto v : g[u]) {
    if (v == f) continue;
    dfs1(v, u);
    siz[u] += siz[v];
    if (siz[v] > siz[hson[u]]) hson[u] = v;
  }
}
 
void dfs2(int u, int cur_ltop) {
  nid[u] = cnt++;
  modify(nid[u], a[u]);
  ltop[u] = cur_ltop;
  if (hson[u]) dfs2(hson[u], cur_ltop);
  for (auto v : g[u]) {
    if (v == fa[u] || v == hson[u]) continue;
    dfs2(v, v);
  }
}
 
int query_path(int u, int v) {
  int res = 0;
  while (ltop[u] != ltop[v]) {
    if (dep[ltop[u]] < dep[ltop[v]]) swap(u, v);
    res = max(res, query(nid[ltop[u]], nid[u] + 1));
    u = fa[ltop[u]];
  }
  if (dep[u] > dep[v]) swap(u, v);
  res = max(res, query(nid[u], nid[v] + 1));
  return res;
}
 
int32_t main() {
  cin.tie(nullptr)->sync_with_stdio(false);
  cin >> n >> m;
  for (int i = 1; i <= n; i++) cin >> a[i];
  for (int i = 1; i < n; i++) {
    cin >> x >> y;
    g[x].push_back(y);
    g[y].push_back(x);
  }
 
  dfs1(1, 0);
  dfs2(1, 1);
 
  while (m--) {
    cin >> op >> x >> y;
    if (op == 1)
      modify(nid[x], y);
    else
      cout << query_path(x, y) << ' ';
  }
  return 0;
}
```

![Desktop View](/assets/img/ac1.png){: w="500" }