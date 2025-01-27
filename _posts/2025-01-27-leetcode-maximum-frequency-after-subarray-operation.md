---
title: LeetCode 3434. Maximum Frequency After Subarray Operation
date: 2025-01-27 00:00:00 +0800
categories: []
tags: [leetcode, interview]     # TAG names should always be lowercase
comments: false
math: true
---

### Problem Statement

[Link](https://leetcode.com/problems/maximum-frequency-after-subarray-operation/description/){:target="\_blank"}

You are given an array nums of length  $$n$$ . You are also given an integer  $$k$$ .

You perform the following operation on nums once:

 - Select a subarray  $$nums[i..j]$$  where  $$0 <= i <= j <= n - 1$$ .

 - Select an integer  $$x$$  and add  $$x$$  to all the elements in  $$nums[i..j]$$ .

Find the maximum frequency of the value  $$k$$  after the operation.

### Example

> **Input:** nums = [1,2,3,4,5,6], k = 1
>
> **Output:** 2
>
> **Explanation:** After adding -5 to nums[2..5], 1 has a frequency of 2 in [1, 2, -2, -1, 0, 1].


### Constraints

 - $$1 <= n == nums.length <= 105$$ .
 - $$1 <= nums[i] <= 50$$ .
 - $$1 <= k <= 50$$ .


---

### Explanation 

Assume that we have found a good subarray, than we should change the most
frequent number  $$p$$ to  $$k$$  for making total number of  $$k$$  maximized.

The value we care is  **(total k's count) + (subarray p's count) - (subarray k's count)** .

Cuz the former term is fixed, than finding the later two terms is the thing we should do.

### Complexity

 - **Time:**  $$O(50n)$$ . Since we should try every possibility within the whole range, 
 each try contributes  $$O(n)$$  and the range size is  $$50$$ .
 - **Space:** $$O(1)$$ . Constant space for memorizing maximum and current value.

### </> Code

```c++
class Solution {
public:
  int maxFrequency(vector<int>& nums, int k) {
    auto find_max_subarray_diff = [&](int target_value) -> int {
      int cur_diff = 0, max_diff = 0;
      for (auto num : nums) {
        if (num == target_value) cur_diff++;
        else if (num == k) cur_diff--;
        max_diff = max(max_diff, cur_diff);
        cur_diff = max(0, cur_diff);
      }
      return max_diff;
      };
    int max_diff = 0;
    for (int i = 1; i <= 50; i++) {
      if (i != k) max_diff = max(max_diff, find_max_subarray_diff(i));
    }
    return max_diff + count(nums.begin(), nums.end(), k);
  }
};
```