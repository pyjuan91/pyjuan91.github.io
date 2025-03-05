---
title: LeetCode 3367. Maximize Sum of Weights after Edge Removals
date: 2025-03-05 00:00:00 +0800
categories: []
tags: [leetcode, interview]     # TAG names should always be lowercase
comments: false
math: true
---

### Problem Statement

[Link](https://leetcode.com/problems/maximize-sum-of-weights-after-edge-removals/description/){:target="\_blank"}

You are given an **undirected, weighted** tree.

Try to remove zero or more edges such that:
 - Each node has an edge with at most **k** other nodes, where **k** is given.
 - The sum of the weights of the remaining edges is maximized.

### Example

> **Input:** edges = [[0,1,4],[0,2,2],[2,3,12],[2,4,6]], k = 2
>
> **Output:** 22
>
> **Explanation:** 
>
> ![](../assets/img/lc3367_explanation.png)


### Constraints

 - $$2 <= n <= 10^5$$ .
 - $$1 <= k <= n - 1$$ .
 - $$1 <= edges[i][2] <= 10^6$$ .


---

### Explanation 

We can solve this problem using **dynamic programming on trees**.

We define the state

$$dp(current node, parent node)$$ state 

with two values: $$v1$$ and $$v2$$.

 - $$v1$$ represents the the maximum achievable sum when the **current node is allowed to connect with at most $$k-1$$ children**.

In this case, one connection slot is reserved for linking the current node to its parent.

 - $$v2$$ represents the maximum achievable sum when the **current node is allowed to connect with up to $$k$$ children**, meaning no connection slot is available for the parent.

--- 

**Transition:**

How to get current state's $$v1$$?

We can use **priority_queue-like** datastrcture to keep track of the previous selecting and regret if we have better choice.

**Or we can firstly sum up the subanswers and regret with sorting improvements.**

We sum up all the children's $$v1$$, and we can have more slots if the selecting children has better $$v2$$.

Then, we have one more slot if calculating $$v2$$.

### Complexity

 - **Time:**  $$O(nlg(n))$$.

 - **Space:** $$O(n)$$.

### </> Code

```c++
class Solution {
 public:
  int64_t maximizeSumOfWeights(vector<vector<int32_t>>& edges, int32_t k) {
    int32_t n = static_cast<int32_t>(edges.size()) + 1;

    // Transform the input to adjacency list.
    vector adj(n, set<pair<int32_t, int32_t>>());
    for (auto& edge : edges) {
      int32_t u = edge[0], v = edge[1], w = edge[2];
      adj[u].insert({v, w});
      adj[v].insert({u, w});
    }

    // DFS the potential states.
    // return two values:
    // v1 means current node only connects at most k-1 children.
    // v2 means current node connects at most k children.
    auto dfs = [&](auto&& self, int32_t cur_node,
                   int32_t parent_node) -> pair<int64_t, int64_t> {
      int64_t res = 0;
      vector<int64_t> improvements;

      for (auto [next_node, weight] : adj[cur_node]) {
        if (next_node == parent_node) continue;
        auto [child_v1, child_v2] = self(self, next_node, cur_node);
        res += child_v2;
        int64_t improvement = max(0L, child_v1 + weight - child_v2);
        improvements.push_back(improvement);
      }

      sort(improvements.rbegin(), improvements.rend());
      int64_t sum_v1 = res, sum_v2;
      for (int i = 0;
           i < (k - 1) and i < static_cast<int32_t>(improvements.size()); i++) {
        sum_v1 += improvements[i];
      }

      if (static_cast<int32_t>(improvements.size()) >= k) {
        sum_v2 = sum_v1 + improvements[k - 1];
      } else {
        sum_v2 = sum_v1;
      }

      return {sum_v1, sum_v2};
    };

    return dfs(dfs, 0, -1).second;
  }
};
```