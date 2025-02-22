---
title: LeetCode 3389. Minimum Operations to Make Character Frequencies Equal
date: 2025-02-22 00:00:00 +0800
categories: []
tags: [leetcode, interview]     # TAG names should always be lowercase
comments: false
math: true
---

### Problem Statement

[Link](https://leetcode.com/problems/minimum-operations-to-make-character-frequencies-equal/description/){:target="\_blank"}

You are given a string $$s$$.

A string $$t$$ is called **good** if all characters of $$t$$ occur the same number of times.

You can perform the following operations **any number of times**:

 - Delete a character from $$s$$.
 - Insert a character in $$s$$.
 - Change a character in $$s$$ to its next letter in the alphabet.

Note that you cannot change $$z$$ to $$a$$ using the third operation.

Return the minimum number of operations required to make $$s$$ good.

### Example

> **Input:** s = "aaabc"
>
> **Output:** 2
>
> **Explanation:** We can make $$s$$ good by applying these operations:
> - Change one occurrence of 'a' to 'b'
> - Insert one occurrence of 'c' into s


### Constraints

 - $$3 <= s.length <= 2 * 10^4$$ .
 - $$s$$ contains only lowercase English letters.


---

### Explanation 

First, we count the actual frequency of all letters.

Next, we apply **dynamic programming** to determine the minimum number of operations required.

Assuming that the desired count is fixed, let $$dp(itemIndex, previousQuota)$$ denote the minimum number of operations needed when processing
the current character, taking into account any deletion quota carried over from previous items.

Consider the following state transitions:

**Case 1: No Modification Needed**

If the current frequency is either equal to the desired count or zero, no modification is required.

**Case 2: Current Frequency Exceeds the Desired Count**

If the current frequency is greater than the desired count, the excess elements can be either deleted or transferred to the next item.

**Case 3: Current Frequency is Less Than the Desired Count (Setting Current to Zero)** 

In this case, all elements are forwarded to the next item.

**Case 4: Current Frequency is Less Than the Desired Count (Setting Current to the Desired Count)** 

Here, we first use the deletion quota carried over from previous items and then perform an insertion operation.

### Complexity

 - **Time:**  $$O(26 * n)$$.

 - **Space:** $$O(26 * n)$$.

### </> Code

```c++
class Solution {
 private:
  static constexpr int kAlphabetSize = 26;

 public:
  int makeStringGood(string s) {
    vector item_freq(kAlphabetSize, 0);
    for (const char& c : s) item_freq[c - 'a']++;
    int max_freq = *max_element(item_freq.begin(), item_freq.end());
    int min_steps = static_cast<int>(s.size()) - max_freq;

    for (int target_freq = 1; target_freq <= max_freq; target_freq++) {
      vector dp(kAlphabetSize, unordered_map<int, int>());
      auto GetMinSteps = [&](auto&& self, int item_id, int prev_quota) -> int {
        if (item_id == kAlphabetSize) return 0;
        if (dp[item_id].count(prev_quota)) return dp[item_id][prev_quota];
        int& res = dp[item_id][prev_quota];
        // Case: No Modification Needed.
        if (item_freq[item_id] == 0 or item_freq[item_id] == target_freq)
          return res = self(self, item_id + 1, 0);
        // Case: Current Frequency Exceeds the Desired Count.
        if (item_freq[item_id] > target_freq)
          return res =
                     item_freq[item_id] - target_freq +
                     self(self, item_id + 1, item_freq[item_id] - target_freq);
        // Case: Current Frequency is Less Than the Desired Count (Setting Current to Zero)
        int kase_make_current_to_zero =
            item_freq[item_id] + self(self, item_id + 1, item_freq[item_id]);
        // Case: Current Frequency is Less Than the Desired Count (Setting Current to the Desired Count)
        int kase_add_current_to_target =
            max(0, target_freq - (item_freq[item_id] + prev_quota)) +
            self(self, item_id + 1, 0);
        return res = min(kase_make_current_to_zero, kase_add_current_to_target);
      };
      min_steps = min(min_steps, GetMinSteps(GetMinSteps, 0, 0));
    }
    return min_steps;
  }
};
```