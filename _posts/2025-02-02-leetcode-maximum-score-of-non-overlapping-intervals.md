---
title: LeetCode 3414. Maximum Score of Non-overlapping Intervals
date: 2025-02-02 00:00:00 +0800
categories: []
tags: [leetcode, interview]     # TAG names should always be lowercase
comments: false
math: true
---

### Problem Statement

[Link](https://leetcode.com/problems/maximum-score-of-non-overlapping-intervals/description/){:target="\_blank"}

You are given an array  $$intervals$$  ,  $$intervals[i] = [LEFT_i, RIGHT_i, WEIGHT_i]$$  .

You can choose up to **4** **non-overlapping** intervals.

Find the maximum sum of those weights. 

If two selected sets have the same sum of weight, return **lexicographically smallest** one.

> An array $$a$$ is lexicographically smaller than an array $$b$$ if in the first position where $$a$$ and $$b$$ differ, array $$a$$ has an element that is less than the corresponding element in $$b$$. If the first $$min(a.length, b.length)$$ elements do not differ, then the shorter array is the lexicographically smaller one.
{: .prompt-info}

### Example

> **Input:** intervals = [[1,3,2],[4,5,2],[1,5,5],[6,9,3],[6,7,1],[8,9,1]]
>
> **Output:** [2,3]
>
> **Explanation:** You can choose the intervals with indices 2, and 3 with respective weights of 5, and 3.


### Constraints

 - $$1 <= intevals.length <= 5 * 10^4$$ .
 - $$1 <= LEFT_i <= RIGHT_i <= 10^9$$ .
 - $$1 <= WEIGHT_i <= 10^9$$ .


---

### Explanation 

The concept behind this problem is **dynamic programming**.

The optimal state at any given step (whether to select the current interval) is determined by
the **previous optimal state**.

**Core Idea**
Define $$dp[i][j]$$ as the best possible state considering:
 - All intervals ending at or before index **i**.
 - Selecting at most **j** itervals.

**Step-by-Step Approach**
1. Since the exact value of left and right boundaries are not important in this problem, we only need to
maintain their relative order. Therefore, we should first discretize them.
2. We need to remember the original indices of the intervals due to the problem constraints. We must create an additional array to store this information.
3. Implement the dynamic programming logic based on the defined state transition.
4. Retrieve and return the optimal selection set.

### Complexity

 - **Time:**  $$O(nlogn)$$ . 
 
 > Dicretizing $$O(nlogn)$$, creating and sorting $$O(nlogn)$$, DP $$O(4n)$$ (each transit $$O(1)$$), retrieving $$O(1)$$.

 - **Space:** $$O(n)$$ .

### </> Code

```c++
class Solution {
private:
  int discrete(vector<vector<int>>& intervals) {
    map<int, int> discrete;
    for (auto& interval : intervals) {
      discrete[interval[0]] = 0;
      discrete[interval[1]] = 0;
    }
    int idx = 0;
    for (auto& [key, value] : discrete) {
      value = idx++;
    }
    for (auto& interval : intervals) {
      interval[0] = discrete[interval[0]];
      interval[1] = discrete[interval[1]];
    }
    return discrete.size();
  }

  bool lexico_smaller(const set<int>& a, const set<int>& b) {
    for (auto [a_idx, b_idx] = pair(a.begin(), b.begin()); a_idx != a.end() and b_idx != b.end(); a_idx++, b_idx++) {
      if (*a_idx < *b_idx) return true;
      if (*a_idx > *b_idx) return false;
    }
    return a.size() < b.size();
  }

  struct Interval {
    int idx, start, end, weight;
    Interval(int idx, int start, int end, int weight) : idx(idx), start(start), end(end), weight(weight) {}
    bool operator<(const Interval& other) const {
      return end < other.end;
    }
  };
public:
  vector<int> maximumWeight(vector<vector<int>>& intervals) {
    int n = discrete(intervals);
    vector<deque<Interval>> interval_objects(4);
    for (int i = 0; i < intervals.size(); i++) {
      for (int j = 1; j <= 4; j++) {
        interval_objects[j - 1].emplace_back(i, intervals[i][0], intervals[i][1], intervals[i][2]);
      }
    }
    for (int i = 0; i < 4; i++) {
      sort(interval_objects[i].begin(), interval_objects[i].end());
    }

    vector dp(n + 1, vector(5, pair(0LL, set<int>())));
    for (int i = 0; i <= n; i++) {
      for (int j = 1; j <= 4; j++) {
        if (i) dp[i][j] = dp[i - 1][j];
        while (!interval_objects[j - 1].empty() and interval_objects[j - 1].front().end == i) {
          auto [idx, start, end, weight] = interval_objects[j - 1].front();
          interval_objects[j - 1].pop_front();
          if ((start ? dp[start - 1][j - 1].first : 0LL) + weight > dp[i][j].first) {
            if (start) dp[i][j] = { dp[start - 1][j - 1].first + weight, dp[start - 1][j - 1].second };
            else dp[i][j] = { weight, {} };
            dp[i][j].second.insert(idx);
          }
          else if ((start ? dp[start - 1][j - 1].first : 0LL) + weight == dp[i][j].first) {
            set<int> new_answer = (start ? dp[start - 1][j - 1].second : set<int>());
            new_answer.insert(idx);
            if (lexico_smaller(new_answer, dp[i][j].second)) {
              dp[i][j].second = new_answer;
            }
          }
        }
      }
    }

    pair<long long, set<int>> answer = { 0LL, set<int>() };
    for (int i = 1; i <= 4; i++) {
      if (dp[n][i].first > answer.first) {
        answer = dp[n][i];
      }
      else if (dp[n][i].first == answer.first) {
        if (lexico_smaller(dp[n][i].second, answer.second)) {
          answer = dp[n][i];
        }
      }
    }
    return vector(answer.second.begin(), answer.second.end());
  }
};
```