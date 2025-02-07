---
title: LeetCode 3410. Maximize Subarray Sum After Removing All Occurrences of One Element
date: 2025-02-07 00:00:00 +0800
categories: []
tags: [leetcode, interview]     # TAG names should always be lowercase
comments: false
math: true
---

### Problem Statement

[Link](https://leetcode.com/problems/maximize-subarray-sum-after-removing-all-occurrences-of-one-element/description/){:target="\_blank"}

You are given an array  $$nums$$  .

You can do the following operation on the array **at most once**:

 - Choose any integer $$x$$ such that $$nums$$ remains non-empty on removing all occurrences of $$x$$ .
 - Remove **all** occurrences of $$x$$ from the array.

Return the maximum subarray sum across all possible resulting arrays.

### Example

> **Input:** nums = [-3,2,-2,-1,3,-2,3]
>
> **Output:** 7
>
> **Explanation:** We can have the following arrays after at most one operation:
> - The original array is nums = [-3, 2, -2, -1, 3, -2, 3]. The maximum subarray sum is 3 + (-2) + 3 = 4.
> - Deleting all occurences of x = -3 results in nums = [2, -2, -1, 3, -2, 3]. The maximum subarray sum is 3 + (-2) + 3 = 4.
> - Deleting all occurences of x = -2 results in nums = [-3, 2, -1, 3, 3]. The maximum subarray sum is 2 + (-1) + 3 + 3 = 7.
> - Deleting all occurences of x = -1 results in nums = [-3, 2, -2, 3, -2, 3]. The maximum subarray sum is 3 + (-2) + 3 = 4.
> - Deleting all occurences of x = 3 results in nums = [-3, 2, -2, -1, -2]. The maximum subarray sum is 2.
>
> The output is max(4, 4, 7, 4, 2) = 7.


### Constraints

 - $$1 <= nums.length <= 10^5$$ .
 - $$-10^6 <= nums[i] <= 10^6$$ .


---

### Explanation 

We should have an clean and comprehensive understanding of **Kadane's Algorithm** to solve this problem.

The main idea is that we need to maintain two things: the current prefix sum and the smallest prefix sum encountered so far.

By subtracting the smallest prefix sum from the current prefix sum as we iterate through each element, we can determine the maximum subarray sum.

In our approach, we can perform an additional operation—deleting certain elements—to further reduce the **original smallest prefix sum** (which can lead to an even better result).

The key implementation detail is using a map to track the **smallest prefix sum** when **deleting a specific value**.

### Complexity

 - **Time:**  $$O(nlogn), O(n)$$ . 
> map: O(nlgn), unordered_map: O(n).

 - **Space:** $$O(n)$$ .

### </> Code

```c++
class Solution {
public:
  long long maxSubarraySum(vector<int>& nums) {
    unordered_map<int, long long> pre_min_when_deleting;
    long long sum = 0;
    long long max_sum = INT64_MIN;
    long long pre_min = 0;
    pre_min_when_deleting[0] = 0;

    for (const auto& num : nums) {
      sum += num;
      max_sum = max(max_sum, sum - pre_min);

      if (num < 0) {
        if (pre_min_when_deleting.count(num)) {
          pre_min_when_deleting[num] = min(pre_min_when_deleting[num], pre_min_when_deleting[0]) + num;
        }
        else {
          pre_min_when_deleting[num] = pre_min_when_deleting[0] + num;
        }
        pre_min = min(pre_min, pre_min_when_deleting[num]);
      }

      pre_min_when_deleting[0] = min(pre_min_when_deleting[0], sum);
      pre_min = min(pre_min, pre_min_when_deleting[0]);
    }

    return max_sum;
  }
};
```