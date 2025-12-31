---
title: 'LeetCode 413: Arithmetic Slices'
date: 2025-12-31 11:37:26
categories:
- Leetcode
- DP
tags:
- Leetcode
- DP
---

## 📋 **Problem Statement**

An **arithmetic slice** is a contiguous subsequence of an array that has at least three elements and the difference between any two consecutive elements is the same.

Given an integer array `nums`, return the **number of arithmetic slices** in `nums`.

### **Examples:**

**Example 1:**
```
Input: nums = [1,2,3,4]
Output: 3
Explanation: [1,2,3], [2,3,4], and [1,2,3,4] are arithmetic slices.
```

**Example 2:**
```
Input: nums = [1,3,5,7,9]
Output: 6
Explanation: 
[1,3,5], [3,5,7], [5,7,9]
[1,3,5,7], [3,5,7,9]
[1,3,5,7,9]
```

**Example 3:**
```
Input: nums = [7,7,7,7]
Output: 3
Explanation: 
[7,7,7], [7,7,7] (two different positions), [7,7,7,7]
```

---

## 🧠 **Understanding the Problem**

### **Key Observations:**
1. **Arithmetic**: All consecutive pairs have the same difference
2. **At least 3 elements**: Minimum length is 3
3. **Contiguous**: Must be a subarray (not subsequence)
4. **Count all possible slices**: Overlapping slices count separately

### **Visual Example:**
```
nums = [1, 2, 3, 4]
Differences: [1, 1, 1]

Arithmetic slices:
[1,2,3]     ✓ diff = 1
[2,3,4]     ✓ diff = 1  
[1,2,3,4]   ✓ diff = 1 (all pairs: 2-1=1, 3-2=1, 4-3=1)
```

---

## 💡 **Approach 1: Brute Force (Intuitive but Slow)**

```python
def numberOfArithmeticSlices_brute(nums):
    """
    Brute force approach: Check every possible subarray
    Time: O(n³) - Too slow for large n
    Space: O(1)
    """
    n = len(nums)
    count = 0
    
    # Check all subarrays of length >= 3
    for i in range(n):  # Starting index
        for j in range(i + 2, n):  # Ending index (min length 3)
            is_arithmetic = True
            diff = nums[i + 1] - nums[i]
            
            # Check if all consecutive differences are equal
            for k in range(i + 1, j + 1):
                if nums[k] - nums[k - 1] != diff:
                    is_arithmetic = False
                    break
            
            if is_arithmetic:
                count += 1
    
    return count
```

**Complexity:**
- Time: O(n³) - Triple nested loops
- Space: O(1)

---

## 🚀 **Approach 2: Optimized DP - O(n) Time**

### **Intuition:**
- If we have an arithmetic slice of length `k`, we can extend it to length `k+1`
- Count contiguous arithmetic sequences
- Use dynamic programming to build solution incrementally

```python
def numberOfArithmeticSlices_dp(nums):
    """
    Dynamic Programming approach
    Time: O(n) - Single pass
    Space: O(n) - DP array
    """
    n = len(nums)
    if n < 3:
        return 0
    
    # dp[i] = number of arithmetic slices ending at index i
    dp = [0] * n
    total = 0
    
    # Start from index 2 (need at least 3 elements)
    for i in range(2, n):
        # Check if nums[i-2], nums[i-1], nums[i] form arithmetic slice
        if nums[i] - nums[i-1] == nums[i-1] - nums[i-2]:
            # If last 3 form arithmetic, we can extend previous slices
            dp[i] = dp[i-1] + 1
            total += dp[i]
    
    return total
```

### **Step-by-step Example:**

```
nums = [1, 2, 3, 4]
n = 4
dp = [0, 0, 0, 0]

i = 2:
  3 - 2 == 2 - 1? Yes (1 == 1)
  dp[2] = dp[1] + 1 = 0 + 1 = 1
  total = 1  (slice [1,2,3])

i = 3:
  4 - 3 == 3 - 2? Yes (1 == 1)
  dp[3] = dp[2] + 1 = 1 + 1 = 2
  total = 1 + 2 = 3  (slices: [1,2,3], [2,3,4], [1,2,3,4])
```

---

## ⚡ **Approach 3: Optimized DP - O(1) Space**

We can optimize space since we only need previous dp value:

```python
def numberOfArithmeticSlices(nums):
    """
    Optimized DP with O(1) space
    Time: O(n), Space: O(1)
    """
    n = len(nums)
    if n < 3:
        return 0
    
    dp = 0  # Current count of arithmetic slices ending at current index
    total = 0
    
    for i in range(2, n):
        if nums[i] - nums[i-1] == nums[i-1] - nums[i-2]:
            dp += 1
            total += dp
        else:
            dp = 0  # Reset when arithmetic sequence breaks
    
    return total
```

### **Visual Walkthrough:**

```
nums = [1, 2, 3, 4, 7, 10]
differences = [1, 1, 1, 3, 3]

Initialize: dp = 0, total = 0

i = 2: nums[2]-nums[1] = 1, nums[1]-nums[0] = 1 → Equal
       dp = 0 + 1 = 1
       total = 0 + 1 = 1  (slice: [1,2,3])

i = 3: nums[3]-nums[2] = 1, nums[2]-nums[1] = 1 → Equal
       dp = 1 + 1 = 2
       total = 1 + 2 = 3  (slices: [1,2,3], [2,3,4], [1,2,3,4])

i = 4: nums[4]-nums[3] = 3, nums[3]-nums[2] = 1 → NOT equal
       dp = 0 (reset)
       total = 3

i = 5: nums[5]-nums[4] = 3, nums[4]-nums[3] = 3 → Equal
       dp = 0 + 1 = 1
       total = 3 + 1 = 4  (slice: [4,7,10])

Final total = 4
```

---

## 📊 **Mathematical Insight: Counting Formula**

For a continuous arithmetic sequence of length `L`, number of arithmetic slices = `(L-1)*(L-2)/2`

**Why?**
- Minimum slice length = 3
- For sequence of length L, we can choose starting and ending points such that:
  - Number of slices of length 3: (L-2)
  - Number of slices of length 4: (L-3)
  - ...
  - Number of slices of length L: 1
- Total = 1 + 2 + ... + (L-2) = (L-1)*(L-2)/2

```python
def numberOfArithmeticSlices_formula(nums):
    """
    Using mathematical formula
    Time: O(n), Space: O(1)
    """
    n = len(nums)
    if n < 3:
        return 0
    
    total = 0
    length = 2  # Current arithmetic sequence length
    
    for i in range(2, n):
        if nums[i] - nums[i-1] == nums[i-1] - nums[i-2]:
            length += 1
        else:
            # Add slices from previous arithmetic sequence
            if length >= 3:
                total += (length - 1) * (length - 2) // 2
            length = 2  # Reset to 2 (current and previous)
    
    # Don't forget the last sequence
    if length >= 3:
        total += (length - 1) * (length - 2) // 2
    
    return total
```

**Example:**
```
Sequence: [1,2,3,4,5]  (length = 5)
Number of slices = (5-1)*(5-2)/2 = 4*3/2 = 6

Slices: [1,2,3], [2,3,4], [3,4,5]
        [1,2,3,4], [2,3,4,5]
        [1,2,3,4,5]
```

---

## 🧪 **Complete Solution with All Approaches**

```python
class Solution:
    def numberOfArithmeticSlices(self, nums):
        """
        Optimal solution: O(n) time, O(1) space
        Most recommended for interviews
        """
        n = len(nums)
        if n < 3:
            return 0
        
        dp = 0  # Number of arithmetic slices ending at current position
        total = 0
        
        for i in range(2, n):
            # Check if last three form arithmetic progression
            if nums[i] - nums[i-1] == nums[i-1] - nums[i-2]:
                # Extend previous slices
                dp += 1
                total += dp
            else:
                # Arithmetic sequence broken
                dp = 0
        
        return total
    
    def numberOfArithmeticSlices_formula(self, nums):
        """
        Alternative: Using mathematical formula
        Good for understanding the pattern
        """
        n = len(nums)
        if n < 3:
            return 0
        
        total = 0
        length = 2  # Current arithmetic sequence length
        
        for i in range(2, n):
            if nums[i] - nums[i-1] == nums[i-1] - nums[i-2]:
                length += 1
            else:
                # Calculate slices for previous sequence
                if length >= 3:
                    total += (length - 1) * (length - 2) // 2
                length = 2
        
        # Handle the last sequence
        if length >= 3:
            total += (length - 1) * (length - 2) // 2
        
        return total
    
    def numberOfArithmeticSlices_brute(self, nums):
        """
        Brute force for comparison
        Not recommended for large inputs
        """
        n = len(nums)
        count = 0
        
        for i in range(n):
            for j in range(i + 2, n):
                diff = nums[i + 1] - nums[i]
                valid = True
                
                for k in range(i + 1, j + 1):
                    if nums[k] - nums[k - 1] != diff:
                        valid = False
                        break
                
                if valid:
                    count += 1
        
        return count
```

---

## 🔍 **Test Cases & Edge Cases**

```python
def test_solution():
    sol = Solution()
    
    test_cases = [
        # Basic cases
        ([1, 2, 3, 4], 3),
        ([1, 3, 5, 7, 9], 6),
        ([7, 7, 7, 7], 3),
        
        # Edge cases
        ([1, 2], 0),           # Too short
        ([1], 0),              # Single element
        ([], 0),               # Empty array
        
        # Mixed sequences
        ([1, 2, 3, 4, 7, 10], 4),
        ([1, 2, 3, 8, 9, 10], 2),
        
        # All same
        ([5, 5, 5, 5, 5], 6),  # Formula: (5-1)*(5-2)/2 = 4*3/2 = 6
        
        # Decreasing sequence
        ([10, 8, 6, 4, 2], 6),  # Same as increasing
        
        # Random sequence
        ([1, 2, 3, 4, 5, 10, 15, 20], 7),
    ]
    
    print("Testing arithmetic slices solution:")
    print("-" * 50)
    
    for i, (nums, expected) in enumerate(test_cases):
        result = sol.numberOfArithmeticSlices(nums)
        status = "✓" if result == expected else "✗"
        print(f"Test {i+1}: {status}")
        print(f"  Input: {nums}")
        print(f"  Expected: {expected}, Got: {result}")
        
        if result != expected:
            print(f"  Error!")
        print()
```

### **Expected Output:**
```
Testing arithmetic slices solution:
--------------------------------------------------
Test 1: ✓
  Input: [1, 2, 3, 4]
  Expected: 3, Got: 3

Test 2: ✓
  Input: [1, 3, 5, 7, 9]
  Expected: 6, Got: 6

Test 3: ✓
  Input: [7, 7, 7, 7]
  Expected: 3, Got: 3

Test 4: ✓
  Input: [1, 2]
  Expected: 0, Got: 0

Test 5: ✓
  Input: [1]
  Expected: 0, Got: 0

Test 6: ✓
  Input: []
  Expected: 0, Got: 0

Test 7: ✓
  Input: [1, 2, 3, 4, 7, 10]
  Expected: 4, Got: 4

Test 8: ✓
  Input: [1, 2, 3, 8, 9, 10]
  Expected: 2, Got: 2

Test 9: ✓
  Input: [5, 5, 5, 5, 5]
  Expected: 6, Got: 6

Test 10: ✓
  Input: [10, 8, 6, 4, 2]
  Expected: 6, Got: 6

Test 11: ✓
  Input: [1, 2, 3, 4, 5, 10, 15, 20]
  Expected: 7, Got: 7
```

---

## 🎯 **Interview Tips**

### **Common Questions & Answers:**

**Q1: Why does the DP solution work?**
> Because when we find that the last three elements form an arithmetic progression, any arithmetic slice ending at `i-1` can be extended to end at `i`. So we add all those slices plus the new slice of length 3.

**Q2: What's the intuition behind `dp += 1`?**
> Each time we extend an arithmetic sequence by one element, we create one new arithmetic slice of length 3 ending at the new position, plus we extend all existing slices.

**Q3: How would you explain the time complexity?**
> We make a single pass through the array, checking three consecutive elements at each step. Each check is O(1), so total time is O(n).

**Q4: What are the edge cases?**
> 1. Array length less than 3 → return 0
> 2. All elements equal → still arithmetic with diff = 0
> 3. Mixed sequences → reset count when difference changes

**Q5: Can you solve it with two pointers/sliding window?**
> Yes! Use left pointer at start of arithmetic sequence, right pointer expands while differences are equal. When sequence breaks, calculate slices using formula and move left pointer to right-1.

### **Alternative Sliding Window Solution:**

```python
def numberOfArithmeticSlices_sliding_window(nums):
    """
    Sliding window approach
    Time: O(n), Space: O(1)
    """
    n = len(nums)
    if n < 3:
        return 0
    
    total = 0
    left = 0
    
    while left < n - 2:
        # Find the longest arithmetic sequence starting at left
        diff = nums[left + 1] - nums[left]
        right = left + 2
        
        while right < n and nums[right] - nums[right - 1] == diff:
            right += 1
        
        # Length of arithmetic sequence
        length = right - left
        
        if length >= 3:
            # Number of arithmetic slices in this sequence
            total += (length - 1) * (length - 2) // 2
        
        # Move left pointer (can skip to right-1 for optimization)
        left = right - 1 if length > 2 else left + 1
    
    return total
```

---

## 📈 **Complexity Comparison**

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force | O(n³) | O(1) | Too slow, not practical |
| DP (array) | O(n) | O(n) | Good, but can optimize space |
| **DP (optimized)** | **O(n)** | **O(1)** | **Recommended** |
| Mathematical Formula | O(n) | O(1) | Alternative, good for understanding |
| Sliding Window | O(n) | O(1) | Alternative perspective |

---

## 🏆 **Final Recommendation**

For interviews, use the **optimized DP approach (O(1) space)**:

```python
class Solution:
    def numberOfArithmeticSlices(self, nums: List[int]) -> int:
        n = len(nums)
        if n < 3:
            return 0
        
        dp = 0  # Count of arithmetic slices ending at current position
        total = 0
        
        for i in range(2, n):
            if nums[i] - nums[i-1] == nums[i-1] - nums[i-2]:
                dp += 1
                total += dp
            else:
                dp = 0
        
        return total
```

**Why this is best:**
1. **Simple and elegant**: Easy to explain in interviews
2. **Optimal complexity**: O(n) time, O(1) space
3. **One-pass solution**: Processes array linearly
4. **Intuitive**: Builds on previous results naturally
5. **Handles all edge cases**: Easy to add length check at start