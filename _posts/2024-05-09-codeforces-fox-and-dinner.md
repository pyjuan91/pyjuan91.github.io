---
title: Codeforces Fox And Dinner
date: 2024-05-09 00:00:00 +0800
categories: []
tags: [codeforces] # TAG names should always be lowercase
comments: false
math: true
---

[題目連結](https://codeforces.com/contest/510/problem/E){:target="\_blank"}

## 題意

給定 $$3<=n<=200$$ 個數字，每個數字 $$2<=x<=10^4$$，尋找分組方法。

1. 每組最少三人最多不限。
2. 每組排成環形，每個環中，任意兩個相鄰的數字和要為質數
3. 每個數字都要恰分配至一組中。

輸出任意合理的分組方法。

---

## 分析

這種每個項目洽在一組的題目，高機率是網路流的 matching 題目（？

但是這個題目的建圖方法好像也不是這麼容易，剛開始看不懂如何轉換為網路流的形狀。

> 觀察到每個數字都至少大於二，這是可以利用的性質
{: .prompt-info}


1. 因為數字最小為二，又兩個數字和為質數，因此必為一奇數一偶數交錯的環！
2. 將題目轉換成奇數偶數的二分圖匹配
3. 一個環內奇數的左手右手各拉著一個偶數，偶數的左右手各拉著一個奇數

要注意的是兩個數不可以構成環，因此源點在給奇數點容量要給兩份，並且檢查最大流是不是恰為 n 即可。

---

## 實作

首先將奇數點和偶數點分開，並分別對原點和匯點給予 2 容量的邊（二容量確保每個環至少為四人）
網路流圖就直接使用[ac-library](https://atcoder.github.io/ac-library/master/document_en/){:target="\_blank"}的模板。

```c++
int n;
cin >> n;
vector<int> a(n + 1), odd_pos, even_pos;
for (int i = 1; i <= n; i++) {
  cin >> a[i];
  if (a[i] % 2)
    odd_pos.push_back(i);
  else
    even_pos.push_back(i);
}

MaxFlowGraph<int> graph(n + 2);
for (auto i : odd_pos) {
  graph.add_edge(0, i, 2);
}
for (auto i : even_pos) {
  graph.add_edge(i, n + 1, 2);
}
```

我們就得到像下面這種圖

![Desktop View](/assets/img/graph0.png){: w="500" }

接著就將可以組合成質數的奇偶對建邊，容量為1。

> 因為數字很小，最大只會到 $$ 2 * 10^4 $$，所以用根號檢查質數不會超時。
{: .prompt-tip}

```c++
  auto is_prime = [&](int x) {
    if (x < 2) return false;
    for (int i = 2; i * i <= x; i++) {
      if (x % i == 0) return false;
    }
    return true;
  };
 
  for (auto i : odd_pos) {
    for (auto j : even_pos) {
      if (is_prime(a[i] + a[j])) {
        graph.add_edge(i, j, 1);
      }
    }
  }
```

就完成建圖了

![Desktop View](/assets/img/graph1.png){: w="500" }

剩下就是檢查是否可行，並輸出任意正確解。
使用 $$ O(n^2) $$ 的方式 $$ dfs $$ 找解。

> 每個點只會被遍歷一次，但是邊數最多有 $$ O((n/2) * (n/2)) == O(n^2) $$ 
{: .prompt-tip}

```c++
#include <bits/stdc++.h>
using namespace std;

namespace internal {

template <class T>
struct simple_queue {
  std::vector<T> payload;
  int pos = 0;
  void reserve(int n) { payload.reserve(n); }
  int size() const { return int(payload.size()) - pos; }
  bool empty() const { return pos == int(payload.size()); }
  void push(const T& t) { payload.push_back(t); }
  T& front() { return payload[pos]; }
  void clear() {
    payload.clear();
    pos = 0;
  }
  void pop() { pos++; }
};

}  // namespace internal

template <class Cap>
struct MaxFlowGraph {
 public:
  MaxFlowGraph() : MaxFlowGraph(0) {}
  explicit MaxFlowGraph(int n) : _n(n), g(n) {}

  int add_edge(int from, int to, Cap cap) {
    assert(0 <= from && from < _n);
    assert(0 <= to && to < _n);
    assert(0 <= cap);
    int m = int(pos.size());
    pos.push_back({from, int(g[from].size())});
    int from_id = int(g[from].size());
    int to_id = int(g[to].size());
    if (from == to) to_id++;
    g[from].push_back(_edge{to, to_id, cap});
    g[to].push_back(_edge{from, from_id, 0});
    return m;
  }

  struct edge {
    int from, to;
    Cap cap, flow;
  };

  edge get_edge(int i) {
    int m = int(pos.size());
    assert(0 <= i && i < m);
    auto _e = g[pos[i].first][pos[i].second];
    auto _re = g[_e.to][_e.rev];
    return edge{pos[i].first, _e.to, _e.cap + _re.cap, _re.cap};
  }
  std::vector<edge> edges() {
    int m = int(pos.size());
    std::vector<edge> result;
    for (int i = 0; i < m; i++) {
      result.push_back(get_edge(i));
    }
    return result;
  }
  void change_edge(int i, Cap new_cap, Cap new_flow) {
    int m = int(pos.size());
    assert(0 <= i && i < m);
    assert(0 <= new_flow && new_flow <= new_cap);
    auto& _e = g[pos[i].first][pos[i].second];
    auto& _re = g[_e.to][_e.rev];
    _e.cap = new_cap - new_flow;
    _re.cap = new_flow;
  }

  Cap flow(int s, int t) { return flow(s, t, std::numeric_limits<Cap>::max()); }
  Cap flow(int s, int t, Cap flow_limit) {
    assert(0 <= s && s < _n);
    assert(0 <= t && t < _n);
    assert(s != t);

    std::vector<int> level(_n), iter(_n);
    internal::simple_queue<int> que;

    auto bfs = [&]() {
      std::fill(level.begin(), level.end(), -1);
      level[s] = 0;
      que.clear();
      que.push(s);
      while (!que.empty()) {
        int v = que.front();
        que.pop();
        for (auto e : g[v]) {
          if (e.cap == 0 || level[e.to] >= 0) continue;
          level[e.to] = level[v] + 1;
          if (e.to == t) return;
          que.push(e.to);
        }
      }
    };
    auto dfs = [&](auto self, int v, Cap up) {
      if (v == s) return up;
      Cap res = 0;
      int level_v = level[v];
      for (int& i = iter[v]; i < int(g[v].size()); i++) {
        _edge& e = g[v][i];
        if (level_v <= level[e.to] || g[e.to][e.rev].cap == 0) continue;
        Cap d = self(self, e.to, std::min(up - res, g[e.to][e.rev].cap));
        if (d <= 0) continue;
        g[v][i].cap += d;
        g[e.to][e.rev].cap -= d;
        res += d;
        if (res == up) return res;
      }
      level[v] = _n;
      return res;
    };

    Cap flow = 0;
    while (flow < flow_limit) {
      bfs();
      if (level[t] == -1) break;
      std::fill(iter.begin(), iter.end(), 0);
      Cap f = dfs(dfs, t, flow_limit - flow);
      if (!f) break;
      flow += f;
    }
    return flow;
  }

  std::vector<bool> min_cut(int s) {
    std::vector<bool> visited(_n);
    internal::simple_queue<int> que;
    que.push(s);
    while (!que.empty()) {
      int p = que.front();
      que.pop();
      visited[p] = true;
      for (auto e : g[p]) {
        if (e.cap && !visited[e.to]) {
          visited[e.to] = true;
          que.push(e.to);
        }
      }
    }
    return visited;
  }

 private:
  int _n;
  struct _edge {
    int to, rev;
    Cap cap;
  };
  std::vector<std::pair<int, int>> pos;
  std::vector<std::vector<_edge>> g;
};

int32_t main() {
  cin.tie(nullptr)->sync_with_stdio(false);

  int n;
  cin >> n;
  vector<int> a(n + 1), odd_pos, even_pos;
  for (int i = 1; i <= n; i++) {
    cin >> a[i];
    if (a[i] % 2)
      odd_pos.push_back(i);
    else
      even_pos.push_back(i);
  }

  MaxFlowGraph<int> graph(n + 2);
  for (auto i : odd_pos) {
    graph.add_edge(0, i, 2);
  }
  for (auto i : even_pos) {
    graph.add_edge(i, n + 1, 2);
  }

  auto is_prime = [&](int x) {
    if (x < 2) return false;
    for (int i = 2; i * i <= x; i++) {
      if (x % i == 0) return false;
    }
    return true;
  };

  for (auto i : odd_pos) {
    for (auto j : even_pos) {
      if (is_prime(a[i] + a[j])) {
        graph.add_edge(i, j, 1);
      }
    }
  }

  if (graph.flow(0, n + 1) != n) {
    cout << "Impossible\n";
    return 0;
  }

  vector<vector<int>> res;
  vector<bool> vis(n + 2);
  auto edges = graph.edges();
  function<void(int, vector<int>&)> dfs = [&](int u, vector<int>& path) {
    vis[u] = true;
    path.push_back(u);
    for (auto edge : edges) {
      if (edge.from == u && edge.flow == 1 && !vis[edge.to]) {
        dfs(edge.to, path);
        return;
      }
      if (edge.to == u && edge.flow == 1 && !vis[edge.from]) {
        dfs(edge.from, path);
        return;
      }
    }
  };
  for (int i = 1; i <= n; i++) {
    if (!vis[i]) {
      vector<int> path;
      dfs(i, path);
      res.push_back(path);
    }
  }

  cout << res.size() << '\n';
  for (auto& path : res) {
    cout << path.size() << ' ';
    for (auto& node : path) {
      cout << node << ' ';
    }
    cout << '\n';
  }
  return 0;
}
```
