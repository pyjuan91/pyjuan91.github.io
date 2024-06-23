---
title: AtCoder Nim
date: 2024-06-20 00:00:00 +0800
categories: []
tags: [atcoder] # TAG names should always be lowercase
comments: false
math: true
---

[Back](https://pyjuan91.github.io/posts/atcoder-plan/)

[題目連結](https://atcoder.jp/contests/abc317/tasks/abc317_f){:target="\_blank"}

## 題意

給定 $$N (10^{18})$$, $$A (10)$$, $$B (10)$$, $$C (10)$$，\
求有多少個 $$tuple(a, b, c)$$ 符合
1. $$a, b, c$$ 分別為 $$A, B, C$$ 的倍數。
2. $$a, b, c$$是正整數且不超過$$N$$， $$1 <= a, b, c <= N$$
3. 三數xor值為零，$$ a \otimes b \otimes c = 0 $$

---

## 分析

看到ABC的大小大概知道要用[數位dp](https://usaco.guide/gold/digit-dp?lang=cpp){:target="\_blank"}，\
但想了20分鐘還是無從下手，感覺有做過 $$xor$$ 類的，有做過倍數類的，但都很不熟😵。\

$$ => $$ 題解局

這波數位dp需要知道
1. Bit 數，因為要 $$xor$$，因此是使用二進位而非十進位。
1. 倍數 -> 用餘數為零來處理。
1. 是否抵達上限 $$N$$，as usual，數位dp老招。

> 三數都不可為零，因此需要特判製作完一個數字時，此數是否為零。\
> 再加三維來判斷。
{: .prompt-info}

至於此時的三數 $$xor$$ 是否為零，則是簡單的貪心。\
每次製作新的 bit 時，都保證三數 $$xor$$ 為零，則最後必為零。

因此製作方法有
1. 全零，$$ bit_a = 0, bit_b = 0, bit_c = 0 $$
1. AB當前位數設一，$$ bit_a = 1, bit_b = 1, bit_c = 0 $$
1. AC當前位數設一，$$ bit_a = 1, bit_b = 0, bit_c = 1 $$
1. BC當前位數設一，$$ bit_a = 0, bit_b = 1, bit_c = 1 $$


---

## 實作

在遞迴函式中，檢查製作完成時是否
1. 各自是倍數
2. 均為正整數



```c++
int digit_dfs(int cur_bit, int amod, int bmod, int cmod,
  int azero, int bzero, int czero, int alim, int blim, int clim) {
  if (cur_bit == -1) {
    return !amod and !bmod and !cmod and !azero and !bzero and !czero;
  }
  int& res = dp[cur_bit][amod][bmod][cmod][azero][bzero][czero][alim][blim][clim];
  if (res != -1) return res;
  
  /*
  recursion part...
  */

  return res;
}
```

遞迴的 part ，我們就任填 $$01$$ 給 $$a, b, c$$，
1. $$lim$$ 表示截至目前的製作，是否都一直貼和上界。\
如果是，則不可填超過當前的上界，否則可隨意填。
2. $$mod$$ 表示各自的餘數。當前填的數，會讓下次遞迴的餘數狀態改變。
3. $$zero$$ 表示當前的數是否還是全填 $$0$$。

如此便可完成此題
```c++
#include <bits/stdc++.h>
#define int long long
using namespace std;

const int kMod = 998244353;
int dp[64][10][10][10][2][2][2][2][2][2];
int n, a, b, c;

int digit_dfs(int cur_bit, int amod, int bmod, int cmod,
  int azero, int bzero, int czero, int alim, int blim, int clim) {
  if (cur_bit == -1) {
    return !amod and !bmod and !cmod and !azero and !bzero and !czero;
  }
  int& res = dp[cur_bit][amod][bmod][cmod][azero][bzero][czero][alim][blim][clim];
  if (res != -1) return res;
  res = 0;

  int lim = (n >> cur_bit) & 1;
  for (int abit = 0; abit <= 1; abit++) {
    for (int bbit = 0; bbit <= 1; bbit++) {
      for (int cbit = 0; cbit <= 1; cbit++) {
        if ((alim and abit) > lim) continue;
        if ((blim and bbit) > lim) continue;
        if ((clim and cbit) > lim) continue;
        if (abit ^ bbit ^ cbit) continue;
        int pamod = (amod + (abit << cur_bit)) % a;
        int pbmod = (bmod + (bbit << cur_bit)) % b;
        int pcmod = (cmod + (cbit << cur_bit)) % c;
        int pazero = azero and !abit;
        int pbzero = bzero and !bbit;
        int pczero = czero and !cbit;
        int palim = alim and abit == lim;
        int pblim = blim and bbit == lim;
        int pclim = clim and cbit == lim;
        res += digit_dfs(cur_bit - 1, pamod, pbmod, pcmod,
          pazero, pbzero, pczero, palim, pblim, pclim);
        res %= kMod;
      }
    }
  }
  return res;
}

int32_t main() {
  cin.tie(nullptr)->sync_with_stdio(false);
  memset(dp, -1LL, sizeof(dp));
  cin >> n >> a >> b >> c;
  cout << digit_dfs(63, 0, 0, 0, 1, 1, 1, 1, 1, 1) << '\n';
  return 0;
}
```

[Back](https://pyjuan91.github.io/posts/atcoder-plan/)