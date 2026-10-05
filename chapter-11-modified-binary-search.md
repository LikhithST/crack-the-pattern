# Chapter 11: Modified Binary Search

## Executive Overview
**Modified Binary Search** extends standard logarithmic search ($O(\log N)$ time, $O(1)$ space) beyond simple value lookups on sorted arrays [183]. It is used when the data structure has been structurally altered (e.g., rotated arrays), contains duplicates, is unbounded, or when searching across a **range of potential answer values** rather than array elements [183, 184, 187].

The fundamental paradigm shift is viewing Binary Search not as a data lookup tool, but as an **optimization framework** for monotonic decision spaces [185, 189].

---

## 1. Major Interview Variations

### Variation 1: Search in Rotated Sorted Array
- **Problem**: A sorted array has been rotated at an unknown pivot (e.g., `[4, 5, 6, 7, 0, 1, 2]`).
- **Core Invariant**: At any midpoint `mid`, **at least one half of the array (left half `[low...mid]` or right half `[mid...high]`) is guaranteed to be strictly sorted** [184].
- **Algorithm**:
  1. Calculate `mid = low + (high - low) // 2`.
  2. Determine which half is properly sorted by comparing `nums[low]` and `nums[mid]` [184].
  3. Check if target lies within the explicit value boundaries of that sorted half [184, 185]:
     - If YES: Narrow search range to that sorted half [184, 185].
     - If NO: Search the opposite (un-sorted/pivot) half [184, 185].

```python
def search_rotated_sorted_array(nums, target):
    low, high = 0, len(nums) - 1
    
    while low <= high:
        mid = low + (high - low) // 2
        if nums[mid] == target:
            return mid
            
        # Left half is sorted [184]
        if nums[low] <= nums[mid]:
            if nums[low] <= target < nums[mid]: # Target lies in left sorted half [184, 185]
                high = mid - 1
            else:
                low = mid + 1
        # Right half is sorted [184]
        else:
            if nums[mid] < target <= nums[high]: # Target lies in right sorted half [184, 185]
                low = mid + 1
            else:
                high = mid - 1
                
    return -1
```

#### Duplicates Edge Case (`nums[low] == nums[mid] == nums[high]`)
When duplicates are present, ambiguity arises because both halves appear identical [186]. The safe fallback is to shrink boundaries (`low += 1`, `high -= 1`), which degrades worst-case runtime to $O(N)$ [186].

---

### Variation 2: Binary Search on Answer / Feasibility Search (Crucial)
- **Problem**: Find the minimum or maximum threshold value satisfying a condition (e.g., minimum ship capacity to deliver packages within $D$ days) [184, 185].
- **Search Space**: The search range `[low, high]` represents **possible answer values**, not array indices [184, 185].
- **Feasibility Function**: A helper function `is_feasible(candidate)` returning boolean `True/False` [185, 188]. The function **must be monotonic**: if `candidate` is feasible, all larger candidates are also feasible [185, 188].

```python
def min_ship_capacity(weights, days):
    # Search space: min capacity is max(weights), max capacity is sum(weights) [185]
    low, high = max(weights), sum(weights)
    result = high
    
    def is_feasible(capacity):
        days_needed = 1
        current_load = 0
        for w in weights:
            if current_load + w > capacity:
                days_needed += 1
                current_load = 0
            current_load += w
        return days_needed <= days # Monotonic check [185, 188]

    while low <= high:
        mid = low + (high - low) // 2
        if is_feasible(mid):
            result = mid      # Valid candidate found; try to find a smaller valid capacity [185]
            high = mid - 1
        else:
            low = mid + 1     # Infeasible; need larger capacity [185]
            
    return result
```

---

### Variation 3: First and Last Occurrence (Boundary Search)
- **Problem**: Find the start and end index of a target value in a sorted array with duplicates [185, 187].
- **Mechanics**:
  - **First Occurrence (`bisect_left`)**: When `nums[mid] == target`, don't stop; set `high = mid - 1` to keep searching left [187].
  - **Last Occurrence (`bisect_right`)**: When `nums[mid] == target`, don't stop; set `low = mid + 1` to keep searching right [187].

---

### Variation 4: Unbounded / Unknown Size Array (Galloping Search)
- **Problem**: Search in a sorted array/stream of unknown or infinite size [187].
- **Algorithm**: Exponentially expand range bounds by doubling index (`high = 1`, then `high *= 2`) until `nums[high] >= target` [187]. Then apply standard Binary Search on `[high // 2, high]` [187].

---

## 2. Trigger Keywords & Decision Rules

### A. Primary Triggers
- Array is sorted or rotated sorted [183, 184].
- Trigger Phrases for Binary Search on Answer: `"minimize the maximum"`, `"maximize the minimum"`, `"smallest capacity"`, `"minimum days"` [185, 188].

### B. Common Bugs & Edge Cases
- **Integer Overflow**: Use `mid = low + (high - low) // 2` instead of `(low + high) // 2` [184].
- **Off-By-One In Termination**: Mixing up `while low <= high:` vs `while low < high:` [183, 184].
