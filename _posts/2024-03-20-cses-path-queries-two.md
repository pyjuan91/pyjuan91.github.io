---
title: CSES Path Queries II (更新的坑人測資)
date: 2024-03-20 00:00:00 +0800
categories: []
tags: [cses]     # TAG names should always be lowercase
comments: false
math: true
---

[題目連結](https://cses.fi/problemset/task/2134/)

## 題意

給一個 $$n$$ 個節點的樹，並需要操作 $$q$$ 個指令 ($$n$$, $$q$$ 都是 $$2e5$$ 等級)
指令有兩種：
1. 單點更新樹的值
2. 找出 $$u$$ 到 $$v$$ 的路徑上，所有點中的最大值

---

這題是`heavy-light decomposition`的模板(?題 (中文叫樹鏈剖分)

雖然是模板題，但是第九筆的測資非常坑人，用最 general 的 template 會 `TLE`

> 第九筆測資應該是judge新增的，因為網路上的一些舊題解都會在第九筆測資TLE (e.g. [USACO Guide 的題解](https://usaco.guide/plat/hld?lang=cpp)  ，不過已向他們發issue 應該很快會更新)
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

int sz[N], p[N], dep[N];
int st[S], id[N], tp[N];

void update(int idx, int val) {
	st[idx += n] = val;
	for (idx /= 2; idx; idx /= 2) st[idx] = max(st[2 * idx], st[2 * idx + 1]);
}

int query(int lo, int hi) {
	int ra = 0, rb = 0;
	for (lo += n, hi += n + 1; lo < hi; lo /= 2, hi /= 2) {
		if (lo & 1) ra = max(ra, st[lo++]);
		if (hi & 1) rb = max(rb, st[--hi]);
	}
	return max(ra, rb);
}

int dfs_sz(int cur, int par) {
	sz[cur] = 1;
	p[cur] = par;
	for (int chi : adj[cur]) {
		if (chi == par) continue;
		dep[chi] = dep[cur] + 1;
		p[chi] = cur;
		sz[cur] += dfs_sz(chi, cur);
	}
	return sz[cur];
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

int path(int x, int y) {
	int ret = 0;
	while (tp[x] != tp[y]) {
		if (dep[tp[x]] < dep[tp[y]]) swap(x, y);
		ret = max(ret, query(id[tp[x]], id[x]));
		x = p[tp[x]];
	}
	if (dep[x] > dep[y]) swap(x, y);
	ret = max(ret, query(id[x], id[y]));
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
			int res = path(a, b);
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

> 感謝網路上各種天才的 tricks，真的有夠香，參考Codeforces上一篇[神文](https://codeforces.com/blog/entry/18051)
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