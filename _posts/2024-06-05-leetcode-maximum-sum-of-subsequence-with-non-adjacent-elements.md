---
title: LeetCode Maximum Sum of Subsequence With Non-adjacent Elements
date: 2024-06-05 00:00:00 +0800
categories: []
tags: [leetcode] # TAG names should always be lowercase
comments: false
math: true
---

[題目連結](https://leetcode.com/problems/maximum-sum-of-subsequence-with-non-adjacent-elements/description/){:target="\_blank"}

## 題意

給定 $$5*10^4$$ 長度的數字序列，以及 $$5*10^4$$ 筆操作。

1. 每一個操作會有兩數字 $$[pos, x]$$，表示將位置為 $$pos$$ 的成員改值為 $$x$$。
2. 每一筆更新後求最大子序列和，且這個序列中不可以有相鄰的元素

將每筆詢問的最大子序列和相加並回傳。

例如，序列為 $$[3, 5, 9]$$ ，操作為 $$[[1, -2], [0, -3]]$$ 。\
則第一次操作後序列為 $$[3, -2, 9]$$ ，不相鄰的最大子序列和為 $$3 + 9 = 12$$。\
第二次操作後序列為 $$[-3, -2, 9]$$ ，不相鄰的最大子序列和為 $$9$$。\
回傳 $$(9 + 12) \% (10^9 + 7) = 21$$

---

## 分析1

如果是單純求最大不相鄰子序列長度的話，可以使用動態規劃，在線性的時間複雜度解決。\
但如果每次修改單個數字後再重求一次答案，將會耗費 $$O(n * q)$$ 的時間複雜度，這是不可接受的。\
可行的方法有分塊[Sqrt Decomposition](https://cp-algorithms.com/data_structures/sqrt_decomposition.html){:target="\_blank"}。\
分別維護每個塊的 ($$head0$$, $$head1$$, $$tail0$$, $$tail1$$ 分別為第一元素，第二元素，倒數第一元素，倒數第二元素)

1. 選 $$head0$$ 且選 $$tail1$$ 的最佳解。
2. 選 $$head1$$ 且選 $$tail0$$ 的最佳解。
3. 選 $$head0$$ 且選 $$tail0$$ 的最佳解。
4. 選 $$head1$$ 且選 $$tail1$$ 的最佳解。

這樣每次更新時需要

1. 更新所在「塊」的那四個值，使用動態規劃重新計算 $$\sqrt{n}$$ 大小的區段，花費 $$O(\sqrt{n})$$。
2. 計算整個大序列的最佳解，一樣再使用一次動態規劃，但考慮總共 $$\sqrt{n}$$ 數量的「塊」，花費 $$O(\sqrt{n})$$。

共有 $$q$$ 筆操作，故總花費 $$O(q * \sqrt{n})$$。

---

## 分析2

更好的時間複雜度為 $$O(q * log(n))$$，則是使用線段樹來維護。

概念上與上方所說的分塊類似，線段樹節點需要紀錄該段落的四個資訊。

```c++
struct SegmentTreeNode {
  int left_index, right_index;
  int64_t subsegment_sum[2][2];
  SegmentTreeNode() {
    left_index = right_index = 0;
    subsegment_sum[0][0] = subsegment_sum[0][1] = subsegment_sum[1][0] = subsegment_sum[1][1] = 0;
  }
};
```

唯一需要考慮的就是如何合併兩個節點（此為兩個段落）。

1. 如果我們選了左段的最末元素，則**不可**選右段的最頭元素。
2. 其餘不限，只需確定所選的**最左**和**最右**元素即可。

因為使用模板化的線段數，因此合併的步驟需要 overload 自訂結構體的加法。

```c++
SegmentTreeNode operator+(const SegmentTreeNode& other) const {
  SegmentTreeNode result;
  result.left_index = left_index;
  result.right_index = other.right_index;
  result.subsegment_sum[0][0] =
      max({subsegment_sum[0][0] + other.subsegment_sum[0][0],
           subsegment_sum[0][1] + other.subsegment_sum[0][0],
           subsegment_sum[0][0] + other.subsegment_sum[1][0]});
  result.subsegment_sum[0][1] =
      max({result.subsegment_sum[0][0],
           subsegment_sum[0][1] + other.subsegment_sum[0][1],
           subsegment_sum[0][0] + other.subsegment_sum[1][1],
           subsegment_sum[0][0], other.subsegment_sum[0][1]});
  result.subsegment_sum[1][0] =
      max({result.subsegment_sum[0][0],
           subsegment_sum[1][0] + other.subsegment_sum[0][0],
           subsegment_sum[1][1] + other.subsegment_sum[0][0],
           subsegment_sum[1][0] + other.subsegment_sum[1][0]});
  result.subsegment_sum[1][1] =
      max({result.subsegment_sum[0][0], result.subsegment_sum[0][1],
           result.subsegment_sum[1][0],
           subsegment_sum[1][1] + other.subsegment_sum[0][1],
           subsegment_sum[1][0] + other.subsegment_sum[1][1],
           subsegment_sum[1][0] + other.subsegment_sum[0][1]});
  return result;
}
```

---

## 實作

有了自訂的結構體後加入模板化的線段樹即可在每次以 $$O(lg(n))$$ 修改元素值以及 $$o(lg(n))$$ 求更新數值後的最佳解。

{% raw %}
```c++
struct SegmentTreeNode {
  int left_index, right_index;
  int64_t subsegment_sum[2][2];
  SegmentTreeNode() {
    left_index = right_index = 0;
    subsegment_sum[0][0] = subsegment_sum[0][1] = subsegment_sum[1][0] =
        subsegment_sum[1][1] = 0;
  }
  SegmentTreeNode(int left_index, int right_index,
                  int64_t subsegment_sum[2][2]) {
    this->left_index = left_index;
    this->right_index = right_index;
    this->subsegment_sum[0][0] = subsegment_sum[0][0];
    this->subsegment_sum[0][1] = subsegment_sum[0][1];
    this->subsegment_sum[1][0] = subsegment_sum[1][0];
    this->subsegment_sum[1][1] = subsegment_sum[1][1];
  }
  SegmentTreeNode operator+(const SegmentTreeNode& other) const {
    SegmentTreeNode result;
    result.left_index = left_index;
    result.right_index = other.right_index;
    result.subsegment_sum[0][0] =
        max({subsegment_sum[0][0] + other.subsegment_sum[0][0],
             subsegment_sum[0][1] + other.subsegment_sum[0][0],
             subsegment_sum[0][0] + other.subsegment_sum[1][0]});
    result.subsegment_sum[0][1] =
        max({result.subsegment_sum[0][0],
             subsegment_sum[0][1] + other.subsegment_sum[0][1],
             subsegment_sum[0][0] + other.subsegment_sum[1][1],
             subsegment_sum[0][0], other.subsegment_sum[0][1]});
    result.subsegment_sum[1][0] =
        max({result.subsegment_sum[0][0],
             subsegment_sum[1][0] + other.subsegment_sum[0][0],
             subsegment_sum[1][1] + other.subsegment_sum[0][0],
             subsegment_sum[1][0] + other.subsegment_sum[1][0]});
    result.subsegment_sum[1][1] =
        max({result.subsegment_sum[0][0], result.subsegment_sum[0][1],
             result.subsegment_sum[1][0],
             subsegment_sum[1][1] + other.subsegment_sum[0][1],
             subsegment_sum[1][0] + other.subsegment_sum[1][1],
             subsegment_sum[1][0] + other.subsegment_sum[0][1]});
    return result;
  }
};

template <typename T>
class SegmentTree {
 private:
  std::vector<T> tree;
  int n;

  void build(int node, int start, int end, const std::vector<T>& arr) {
    if (start == end) {
      tree[node] = arr[start];
    } else {
      int mid = (start + end) / 2;
      build(2 * node, start, mid, arr);
      build(2 * node + 1, mid + 1, end, arr);
      tree[node] = tree[2 * node] + tree[2 * node + 1];
    }
  }

  void update(int node, int start, int end, int idx, T val) {
    if (start == end) {
      // Leaf node
      tree[node] = val;
    } else {
      int mid = (start + end) / 2;
      if (start <= idx && idx <= mid) {
        // If idx is in the left child, recurse on the left child
        update(2 * node, start, mid, idx, val);
      } else {
        // If idx is in the right child, recurse on the right child
        update(2 * node + 1, mid + 1, end, idx, val);
      }
      // Internal node will have the sum of both of its children
      tree[node] = tree[2 * node] + tree[2 * node + 1];
    }
  }

  T query(int node, int start, int end, int l, int r) {
    if (r < start || end < l) {
      // range represented by a node is completely outside the given range
      return T();
    }
    if (l <= start && end <= r) {
      // range represented by a node is completely inside the given range
      return tree[node];
    }
    // range represented by a node is partially inside and partially outside the
    // given range
    int mid = (start + end) / 2;
    T p1 = query(2 * node, start, mid, l, r);
    T p2 = query(2 * node + 1, mid + 1, end, l, r);
    return p1 + p2;
  }

 public:
  SegmentTree(const std::vector<T>& arr) {
    n = arr.size();
    tree.resize(4 * n);
    build(1, 0, n - 1, arr);
  }

  void update(int idx, T val) { update(1, 0, n - 1, idx, val); }

  T query(int l, int r) { return query(1, 0, n - 1, l, r); }
};

class Solution {
 public:
  int maximumSumSubsequence(vector<int>& nums, vector<vector<int>>& queries) {
    int n = nums.size();
    vector<SegmentTreeNode> arr(n);
    for (int i = 0; i < n; i++) {
      int64_t subsegment_sum[2][2] = {{0, 0}, {0, nums[i]}};
      arr[i] = SegmentTreeNode(i, i, subsegment_sum);
    }
    SegmentTree<SegmentTreeNode> st(arr);
    int64_t result = 0;
    const int64_t kMod = 1e9 + 7;
    for (const auto& query : queries) {
      int pos = query[0], val = query[1];
      int64_t subsegment_sum[2][2] = {{0, 0}, {0, val}};
      st.update(pos, SegmentTreeNode(pos, pos, subsegment_sum));
      auto node = st.query(0, n - 1);
      int64_t max_sum =
          max({node.subsegment_sum[0][0], node.subsegment_sum[0][1],
               node.subsegment_sum[1][0], node.subsegment_sum[1][1]});
      result = (result + max_sum) % kMod;
    }
    return (int)result;
  }
};
```
{% endraw %}

![Desktop View](/assets/img/leetcode1.png){: w="1000" }

---