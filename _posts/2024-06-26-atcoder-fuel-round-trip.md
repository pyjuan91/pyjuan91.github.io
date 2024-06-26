---
title: AtCoder Fuel Round Trip
date: 2024-06-26 00:00:00 +0800
categories: []
tags: [atcoder] # TAG names should always be lowercase
comments: false
math: true
---

[Back](https://pyjuan91.github.io/posts/atcoder-plan/)

[題目連結](https://atcoder.jp/contests/abc320/tasks/abc320_f){:target="\_blank"}

## 題意

有 $$N <= 300$$ 個座標，前 $$N-1$$ 個座標有加油站。\
現在要你
1. 從座標 $$0$$ 旅行到座標 $$N-1$$ 再回到原點。
2. 每個加油站不論去回程，只能加一次油。
3. 若要買油，必須付 $$P$$ 元，獲得 $$F$$ 升油，多餘的需丟棄。

求如何加油花費最少錢。

---

## 分析

這種加油題目算是 leetcode 動態規劃題的常客。\
不過因為去程加不加油不是這麼單純的事情，會影響到回程。\
因此我們需要多一維度來紀錄**每次做決定時會對狀態造成的影響**。

考慮狀態動態規劃\
$$dp[i][j][k]$$ 表示目前在第 $$i$$ 個車站，準備了 $$j$$ 升油給去程，準備了 $$k$$ 升油給回程。

因此每次可以做的決定有
1. 不論去回程都略過。
2. 為去程加油。
3. 為回程加油。

寫出轉移方法後將超出邊界的 state 都給予非常貴的價格，\
若最後答案不合理的貴，表示無解。


---

## 實作

```c++
#include <bits/stdc++.h>
#define int long long
using namespace std;

const int kMax = 301;
int n, cap;
int x[kMax], price[kMax], fuel[kMax];
int dp[kMax][kMax][kMax];

int zuha(int at_station, int fuel_out, int fuel_back) {
  if (at_station == n) {
    if (fuel_out >= fuel_back) return dp[at_station][fuel_out][fuel_back] = 0;
    else {
      return dp[at_station][fuel_out][fuel_back] = 1e18;
    }
  }
  if (dp[at_station][fuel_out][fuel_back] != -1) {
    return dp[at_station][fuel_out][fuel_back];
  }
  int ans = 1e18;
  int dist = x[at_station + 1] - x[at_station];
  // do nothing neither going out nor back
  if (fuel_out >= dist and fuel_back + dist <= cap) {
    ans = min(ans, zuha(at_station + 1, fuel_out - dist, fuel_back + dist));
  }
  // this station fuel when going out
  if (fuel_back + dist <= cap and min(cap, fuel_out + fuel[at_station]) >= dist) {
    ans = min(ans, price[at_station] +
      zuha(at_station + 1, min(cap, fuel_out + fuel[at_station]) - dist, fuel_back + dist));
  }
  // this station fuel when going back
  if (fuel_out >= dist and max(0LL, fuel_back - fuel[at_station]) + dist <= cap) {
    ans = min(ans, price[at_station] +
      zuha(at_station + 1, fuel_out - dist, max(0LL, fuel_back - fuel[at_station]) + dist));
  }
  return dp[at_station][fuel_out][fuel_back] = ans;
}

int32_t main() {
  cin.tie(nullptr)->sync_with_stdio(false);
  memset(dp, -1LL, sizeof(dp));
  cin >> n >> cap;
  for (int i = 1; i <= n; i++) cin >> x[i];
  for (int i = 1; i < n; i++) cin >> price[i] >> fuel[i];

  int ans = 1e18;
  for (int i = 0; i <= cap; i++) ans = min(ans, zuha(0, cap, i));
  cout << (ans == 1e18 ? -1 : ans) << '\n';
  return 0;
}
```

[Back](https://pyjuan91.github.io/posts/atcoder-plan/)