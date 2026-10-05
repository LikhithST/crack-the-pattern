# Chapter 3: Two Pointers

## Executive Overview
The **Two Pointers** pattern is often referred to as the "Swiss Army knife" of coding interview patterns because of its versatility across a wide range of problem domains [33]. It optimizes brute-force $O(N^2)$ or $O(N^3)$ nested-loop algorithms into **$O(N)$ time and $O(1)$ auxiliary space** by leveraging spatial relationships and monotonicity [33, 34].

Unlike Sliding Window (which tracks the contiguous state *between* pointers), Two Pointers focuses on **comparing or combining elements at specific pointer positions** [27, 33].

---

## 1. The Three Primary Configurations

```
1. CONVERGING CONFIGURATION (Opposite Ends)
   [ left ➔                            right ]
   • Used for: Sorted arrays, pair/triplet sum search, container with most water [33, 34].

2. PARALLEL CONFIGURATION (Two Separate Sequences)
   Array A: [ ptr1 ➔ ]
   Array B: [ ptr2 ➔ ]
   • Used for: Merging sorted arrays, set intersections, 2-sequence comparisons [35].

3. PARTITION CONFIGURATION (Same Array, Multiple Regions)
   [ low... ➔ | mid... ➔ |  high... ]
   • Used for: In-place sorting, Dutch National Flag, QuickSelect pivot partitioning [38, 39].
```

---

## 2. Deep Dive into Configurations & Templates

### Configuration A: Converging Two Pointers
- **Precondition**: Input array is **sorted** (or can be sorted without violating problem constraints) [34, 37].
- **Monotonic Principle**: In a sorted array, moving `left` forward increases the value at `left`, while moving `right` backward decreases the value at `right` [34]. This monotonic property guarantees that every comparison makes a definitive decision to eliminate candidate pairs [34].

```python
def converging_two_pointers(sorted_arr, target):
    left = 0
    right = len(sorted_arr) - 1
    
    while left < right:
        current_val = sorted_arr[left] + sorted_arr[right]
        
        if current_val == target:
            return [left, right] # Pair found
        elif current_val < target:
            left += 1  # Need a larger sum -> move left pointer forward
        else:
            right -= 1 # Need a smaller sum -> move right pointer backward
            
    return [-1, -1]
```

### Extending to Higher Dimensions: 3Sum, 4Sum, K-Sum
To find triplets or quadruplets satisfying a target sum:
1. Sort the array in $O(N \log N)$ time [34].
2. Use outer loops to fix $K - 2$ elements [35].
3. Apply Converging Two Pointers on the remaining sub-array [35].
4. **Complexity**: Reduces $O(N^K)$ brute-force to **$O(N^{K-1})$** time with $O(1)$ extra space [35].

#### Critical Rule: Duplicate Handling
When returning unique triplets/tuples, you **must explicitly skip duplicates** after processing a valid match [38]:
```python
# After finding a valid pair and recording it:
while left < right and sorted_arr[left] == sorted_arr[left + 1]:
    left += 1  # Skip identical values on left
while left < right and sorted_arr[right] == sorted_arr[right - 1]:
    right -= 1 # Skip identical values on right
left += 1
right -= 1
```

---

### Configuration B: Parallel Two Pointers
- **Domain**: Processing two independent sorted sequences simultaneously [35].
- **Core Mechanism**: Maintain `ptr1` for Array A and `ptr2` for Array B. Advance whichever pointer points to the smaller element to build merged output in $O(N + M)$ time [35].
- **Asymmetry Handling**: When one sequence is exhausted before the other, an explicit cleanup loop must append all remaining elements from the unexhausted sequence [40].

```python
def merge_sorted_arrays(arr1, arr2):
    p1, p2 = 0, 0
    result = []
    
    while p1 < len(arr1) and p2 < len(arr2):
        if arr1[p1] <= arr2[p2]:
            result.append(arr1[p1])
            p1 += 1
        else:
            result.append(arr2[p2])
            p2 += 1
            
    # Cleanup remaining elements (Asymmetry Handling)
    while p1 < len(arr1):
        result.append(arr1[p1]); p1 += 1
    while p2 < len(arr2):
        result.append(arr2[p2]); p2 += 1
        
    return result
```

---

### Configuration C: Partition Variant (Dutch National Flag)
- **Domain**: Rearranging elements of an unsorted array in-place into distinct categories (e.g., values equal to 0, 1, 2) [39].
- **Mechanics**: Employs 3 pointers (`low`, `mid`, `high`) to partition the array into three contiguous regions in a single pass [39].

---

## 3. Decision Framework & Trigger Keywords

### A. Primary Triggers (When to Use)
- **Input**: Linear data structures (array, string, sequence) [36].
- **Goal**: Compare, pair, combine, or partition elements from different positions [36].
- **Keywords**: `"pair"`, `"triplet"`, `"two sum"`, `"sorted array"`, `"merge two lists"`, `"partition"`, `"difference"` [36].

### B. Common Traps & Counter-Indicators (When NOT to Use)
1. **Unsorted Array without Sorting Permitted**: Converging two pointers **fails completely on unsorted input** [37]. If sorting is prohibited or index order must be preserved, use **Hash Map (Two Sum)** instead [37].
2. **Non-Contiguous Combinations**: If the problem requires examining all possible subsets or combinations without structural monotonicity, use **Backtracking / Subsets** [37].
3. **Sliding Window Confusion**: If the problem requires tracking a running window aggregate (sum, character frequency count), use **Sliding Window** instead of Two Pointers [27].

---

## 4. Edge Cases & Debugging Protocols

### A. High-Frequency Edge Cases
- **Duplicate Elements**: Forgetting to skip duplicate values leads to duplicate output tuples (e.g., duplicate 3Sum triplets) [38].
- **Exhausted Array Asymmetry**: In parallel configuration, failing to flush remaining elements from the longer array [40].
- **Pointer Crossing & Infinite Loops**: Failing to advance pointers inside conditional branches or missing loop termination conditions (`left < right` vs `left <= right`) [34, 39].

### B. Debugging Protocol
1. **Trace Monotonicity**: Write down sample array values and verify that moving `left` increases the computed metric and moving `right` decreases it [34, 39].
2. **Hand Simulation**: Trace small test cases (3-4 elements) step-by-step on paper, tracking exact pointer indices [39].
3. **Branch Coverage**: Ensure pointer advancement (`left++` or `right--`) occurs in *every* branch of the conditional statement [39, 40].
