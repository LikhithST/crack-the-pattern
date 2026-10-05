# Chapter 6: Cyclic Sort

## Executive Overview
**Cyclic Sort** is a highly specialized, in-place sorting technique that operates in **$O(N)$ time and $O(1)$ auxiliary space** [59]. It is the optimal pattern for problems involving arrays containing numbers in a contiguous range (typically $1$ to $N$ or $0$ to $N$) where elements are missing, duplicated, or out of position [59].

The core insight is **value-to-index mapping**: when numbers are directly related to array indices, each element can be placed into its target index through a series of swaps [59]. Once placed, missing or duplicate values become immediately visible via a single linear scan [59].

---

## 1. Recognition Triggers & Preconditions

### A. Core Criteria
1. **Array Range**: Input array of size $N$ contains integers strictly within range $[1, N]$ or $[0, N-1]$ [59].
2. **Missing/Duplicate Query**: Asked to find missing numbers, duplicate numbers, or the smallest missing positive integer [59, 60].
3. **Strict Space Constraint**: $O(1)$ auxiliary space is explicitly required [59].

```
Index Mapping Rule (1-indexed values in 0-indexed array):
Value  'val'  belongs at Index  'val - 1'  [59].

Example Array:  [3, 1, 5, 4, 2]
Target State:  [1, 2, 3, 4, 5] (where nums[i] == i + 1) [59]
```

---

## 2. Universal Code Template & Non-Negotiable Rules

### The Golden Rule of Index Advancement
When you swap `nums[i]` with `nums[correct_idx]`, **DO NOT advance `i`** [61]. The element newly placed at index `i` came from elsewhere and may *also* be out of position [61]. You must re-evaluate index `i` until it contains the correct element before calling `i += 1` [59, 61].

```python
def cyclic_sort_template(nums):
    i = 0
    while i < len(nums):
        correct_idx = nums[i] - 1  # 1-based indexing formula [59]
        
        # Check if element is valid AND not already in its correct target position
        if 1 <= nums[i] <= len(nums) and nums[i] != nums[correct_idx]:
            # Swap nums[i] to its correct home position
            nums[i], nums[correct_idx] = nums[correct_idx], nums[i] [59]
            # NOTICE: Index 'i' is NOT incremented here! [61]
        else:
            # Advance ONLY when nums[i] is already in correct position or out of range
            i += 1 [59]
            
    return nums
```

### Amortized $O(N)$ Runtime Proof
Although index `i` is not incremented on swap, every swap places **at least one number into its permanent final position** [59]. An array of length $N$ can undergo at most $N - 1$ swaps. Thus, total operations are bounded by $2N$, guaranteeing $O(N)$ time complexity [59].

---

## 3. High-Frequency Variations

### Variation 1: Find the Missing Number
- **Problem**: Array of $N$ numbers containing distinct values from $0$ to $N$. One number is missing.
- **Algorithm**:
  1. Cyclic sort numbers where value `val` maps to index `val`.
  2. Post-sort scan: Return the first index `i` where `nums[i] != i` [59]. If all match, $N$ is missing [59].

### Variation 2: Find All Duplicates / Disappeared Numbers
- **Problem**: Array of $N$ numbers containing range $[1, N]$. Some elements appear twice, others appear once.
- **Algorithm**:
  1. Run Cyclic Sort [59]. Duplicate numbers will fail the swap condition `nums[i] != nums[correct_idx]` because target index `correct_idx` already contains `nums[i]` [61].
  2. Post-sort scan: Iterate $i$ from $0$ to $N-1$.
     - If `nums[i] != i + 1`: `nums[i]` is a **duplicate**, and `i + 1` is a **missing number** [59, 60].

### Variation 3: First Missing Positive (Hard)
- **Problem**: Unsorted integer array. Find the smallest missing positive integer in $O(N)$ time and $O(1)$ space.
- **Algorithm**:
  1. Ignore non-positive numbers ($\le 0$) and numbers $> N$ during swaps (`1 <= nums[i] <= N`) [61].
  2. Cyclic sort valid positive integers to target indices `nums[i] - 1` [59].
  3. Post-sort scan: First index where `nums[i] != i + 1` identifies the first missing positive `i + 1` [61].

---

## 4. Edge Cases & Debugging Framework

### A. High-Frequency Implementation Bugs
1. **Infinite Loop on Duplicates**: Swapping without checking `nums[i] != nums[correct_idx]` causes infinite swapping when duplicate values try to swap with each other [61].
2. **Premature Index Advancement**: Calling `i += 1` inside the swap block skips checking newly arrived elements [61].
3. **Index Out of Bounds / Off-by-One**: Mixing up $0$-indexed (`nums[i] - 1`) vs $1$-indexed mapping [61].

### B. 3-Point Debugging Protocol
1. **Verify Swap Guard**: Ensure condition reads `nums[i] != nums[nums[i] - 1]`, preventing duplicate swaps [61].
2. **Check Index Loop Variable**: Verify `i += 1` executes strictly in the `else` branch [61].
3. **Validate Range Filters**: Verify numbers outside $[1, N]$ are ignored and bypassed gracefully [61].
