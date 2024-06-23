---
title: AtCoder Zero or One
date: 2024-06-23 00:00:00 +0800
categories: []
tags: [atcoder] # TAG names should always be lowercase
comments: false
math: true
---

[題目連結](https://atcoder.jp/contests/abc293/tasks/abc293_f){:target="\_blank"}

## 題意

給定 $$2<=N<=10^{18}$$ 。求共有多少個不同的 $$base$$，\
使得將 $$N$$ 轉換為 $$base$$ 進位時，每個位數均為 $$0$$ 或 $$1$$ ？

---

## 分析

一樣想20分鐘想不出來🥲，想了暴搜或是遞迴的方法，都會大超時。

-> 題解局

對於不同 level 的 $$base$$ 使用不同的方式檢查。（算是數論分塊？）

1. 若 $$base$$ 很小，因為產生的位數很多（二進位最多63位），因此暴力檢查\
$$N$$ 轉換為 $$base$$ 進位是否都是 $$0, 1$$ 。
2. 若 $$base$$ 很大，因為產生的位數很少（一千進位最多7位），因此使用 $$bitmask$$ 這七位數\
再二分搜索是否有合理的 $$base$$ 可以產生 $$N$$。

---

## 實作

檢查 $$N$$ 轉換為 $$base$$ 時，是否都是 $$0, 1$$。

Time complexity: $$O(log(n))$$

```c++
bool check_base_match(int n, int base) {
  while (n) {
    if (n % base > 1) return false;
    n /= base;
  }
  return true;
}
```

---

根據 $$mask$$ 和 猜測的 $$base$$ 轉換為十進位數。\
因為 $$mask$$ 最多只有二進位七位，因此此function可當 $$O(1)$$。

```c++
int transform(int base, int mask) {
  __uint128_t res = 0, p = 1;
  while (mask) {
    res += p * (mask % 2);
    if (res > 1e18) return -1;
    mask /= 2;
    p *= base;
  }
  return res;
}
```

---

猜測 $$N$$ 是由多大的 $$base$$ 製作。

Time complexity: $$O(lg(n))$$

```c++
bool check_mask_match(int n, int mask) {
  int left = 1001, right = 1e18, mid;
  while (left <= right) {
    mid = left + (right - left) / 2;
    int res = transform(mid, mask);
    if (res == -1 || res > n) right = mid - 1;
    else if (res < n) left = mid + 1;
    else return true;
  }
  return false;
}
```

---

解題函式，測 1000 以下的 $$base$$ 用暴搜。
測 1000 以上的 $$base$$ 用 bitmask + binary search。

Time complexity: $$O(1000 * lg(n)) + O(1^7 * lg(n))$$

```c++
int zuha(int n) {
  int res = 0;
  for (int i = 2; i <= 1000; i++) {
    if (check_base_match(n, i)) res++;
  }
  for (int mask = 1; mask < (1 << 6); mask++) {
    if (check_mask_match(n, mask)) res++;
  }
  return res;
}
```

---

最多有 1000 筆測資，因此約為\
$$O(1000 * 2000 * lg (1^{18})) = O(128 * 10^{6})$$

```c++
#include <bits/stdc++.h>
#define int long long
using namespace std;

bool check_base_match(int n, int base) {
  while (n) {
    if (n % base > 1) return false;
    n /= base;
  }
  return true;
}

int transform(int base, int mask) {
  __uint128_t res = 0, p = 1;
  while (mask) {
    res += p * (mask % 2);
    if (res > 1e18) return -1;
    mask /= 2;
    p *= base;
  }
  return res;
}

bool check_mask_match(int n, int mask) {
  int left = 1001, right = 1e18, mid;
  while (left <= right) {
    mid = left + (right - left) / 2;
    int res = transform(mid, mask);
    if (res == -1 || res > n) right = mid - 1;
    else if (res < n) left = mid + 1;
    else return true;
  }
  return false;
}

int zuha(int n) {
  int res = 0;
  for (int i = 2; i <= 1000; i++) {
    if (check_base_match(n, i)) res++;
  }
  for (int mask = 1; mask < (1 << 6); mask++) {
    if (check_mask_match(n, mask)) res++;
  }
  return res;
}

int32_t main() {
  cin.tie(nullptr)->sync_with_stdio(false);
  int t, n;
  cin >> t;
  while (t--) {
    cin >> n;
    cout << zuha(n) << '\n';
  }
  return 0;
}
```