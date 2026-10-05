# Chapter 2: Sliding Window

## Executive Overview
The **Sliding Window** pattern is one of the most foundational and frequently tested techniques in technical interviews [21, 26]. It converts nested loop brute-force algorithms—which examine all possible contiguous subarrays or substrings in $O(N^2)$ or $O(N^3)$ time—into optimized **$O(N)$ linear-time passes** [21, 23]. 

The core mechanism involves maintaining a contiguous segment (window) bounded by two pointers (`left` and `right`) that traverse the linear data structure from left to right [21].

---

## 1. Core Mental Model & Mechanics

### A. Dynamic Pointer Movement
- **`right` Pointer**: Expands the window by moving forward one step at a time, incorporating the new element into the window's running aggregate state [21].
- **`left` Pointer**: Contracts the window by moving forward when specific conditions are met, removing elements from the running aggregate state [22].

### B. Amortized $O(N)$ Complexity Analysis
Although sliding window solutions contain nested loops (`while` loop inside a `for`/`while` loop), the time complexity remains strictly **$O(N)$** [30, 31]:
- Each element is added to the window by the `right` pointer **at most once** [31].
- Each element is removed from the window by the `left` pointer **at most once** [31].
- Total operations across the entire algorithm execution are bounded by $2N$, guaranteeing $O(N)$ amortized runtime [31].

---

## 2. The Two Primary Categories

Determining **when to shrink the window** is the critical decision point that differentiates correct implementations from subtle bugs [22].

```
               ┌────────────────────────────────────────┐
               │    Is the target optimal condition?    │
               └───────────────────┬────────────────────┘
                                   │
         ┌─────────────────────────┴─────────────────────────┐
         ▼                                                   ▼
┌─────────────────────────────────┐         ┌─────────────────────────────────┐
│         LONGEST VALID           │         │         SHORTEST VALID          │
├─────────────────────────────────┤         ├─────────────────────────────────┤
│ • Goal: Maximize window size.   │         │ • Goal: Minimize window size.   │
│ • Shrink rule: Shrink ONLY when │         │ • Shrink rule: Shrink AS SOON   │
│   window becomes INVALID [22].  │         │   AS window becomes VALID [22]. │
│ • Update result: When window is │         │ • Update result: When window is │
│   VALID [22].                   │         │   VALID (before shrinking) [23].│
└─────────────────────────────────┘         └─────────────────────────────────┘
```

### Category 1: Longest Valid Subarray / Substring
- **Objective**: Find the maximum length contiguous segment satisfying a condition (e.g., longest substring with at most $K$ distinct characters) [22, 29].
- **Shrink Decision**: Shrink the window via `left++` **only when the window becomes invalid** [22].
- **Reasoning**: You want to expand as much as possible; you only contract when forced to restore validity [22].

### Category 2: Shortest Valid Subarray / Substring
- **Objective**: Find the minimum length contiguous segment satisfying a condition (e.g., minimum size subarray with sum $\ge$ target) [22, 23].
- **Shrink Decision**: Shrink the window via `left++` **as soon as the window becomes valid** [22, 23].
- **Reasoning**: Once valid, any further expansion makes the window longer (worse). You must immediately contract from the left to search for a smaller valid candidate [23].

---

## 3. Major Variations & Advanced Configurations

### A. Fixed-Size Window
- **Use Case**: Problems asking for calculations over every contiguous segment of exact length $K$ (e.g., maximum sum subarray of size $K$, moving averages) [24].
- **Mechanics**: Advance `right` until window size equals $K$. From then on, slide both `left` and `right` by 1 position simultaneously without a conditional validity check [24].

### B. Monotonic Deque Sliding Window (Max/Min Tracking)
- **Use Case**: Finding the maximum or minimum element within a sliding window in $O(N)$ total time [24, 31].
- **Mechanics**:
  - Maintain a **Double-Ended Queue (Deque)** storing **indices** (not raw values) [31].
  - Maintain indices in **decreasing order of value** for a Maximum Deque (or increasing for Minimum Deque) [31].
  - When adding element at `right`: Pop indices from the back whose values are $\le$ current value (they can never be the maximum again) [31].
  - When sliding `left`: Remove indices from the front if they fall outside the current window boundaries (`deque.front() < left`) [31, 32].
  - The front of the deque always holds the index of the maximum element in $O(1)$ access time [32].

### C. Multi-Constraint Windows
- **Use Case**: Windows requiring simultaneous constraints (e.g., sum constraint AND uniqueness constraint AND length constraint) [28].
- **Strategy**: Prioritize constraint resolution. Resolve simple frequency/uniqueness violations first before evaluating complex aggregate constraints [28].

---

## 4. Universal Code Template & Workflow

### Skeleton-First Approach
Never write sliding window logic without setting up the skeleton first [30]. Draft the loop structure, pointer initialization, and result update before filling in state specifics [30].

```python
def sliding_window_template(arr, target):
    left = 0
    running_state = ... # Initialize state (e.g., int sum, hash map count)
    best_result = default_value # max_len = 0 or min_len = float('inf')
    
    for right in range(len(arr)):
        # 1. Expand window: Add element at 'right' to state
        update_state_add(running_state, arr[right])
        
        # 2. Check validity and shrink window
        # For SHORTEST valid: while is_valid(running_state):
        # For LONGEST valid:  while not is_valid(running_state):
        while condition_triggered(running_state):
            # If tracking Shortest Valid, update result here
            # best_result = min(best_result, right - left + 1)
            
            # Remove element at 'left' from state and contract
            update_state_remove(running_state, arr[left])
            left += 1
            
        # If tracking Longest Valid, update result here (when valid)
        # best_result = max(best_result, right - left + 1)
        
    return best_result
```

---

## 5. When to Use vs. When NOT to Use

### A. Primary Triggers (When to Use)
1. **Contiguous Segments**: Explicit keywords like "contiguous subarray", "substring", "rolling window", or "moving average" [21, 24, 28].
2. **Running Aggregates**: Need to compute cumulative sums, character frequencies, or rolling max/min over linear data [24, 27].

### B. Common Traps & Counter-Indicators (When NOT to Use)
1. **Non-Contiguous Subsequences or Subsets**: If elements can be selected out of order or non-adjacently, Sliding Window **will not work** [24, 25]. Use Subsets/Backtracking or DP instead [25].
2. **Order-Dependent Sequences Inside Window**: If validity depends on the exact relative sequence/ordering of elements within the window, sliding window fails because it treats elements in the window as an unordered collection/aggregate [25].
3. **Sliding Window vs. Two Pointers Distinction**:
   - **Sliding Window**: Focuses on the **contiguous state/elements between pointers** [27].
   - **Two Pointers**: Focuses on **comparing elements at the exact pointer positions** (e.g., sorted array pair sums) [27].
4. **Avoid Heaps Inside Sliding Window**: Do not insert a Min/Max Heap inside a sliding window unless strictly necessary [25]. Heap operations introduce $O(\log K)$ overhead per step, defeating the linear $O(N)$ goal [25]. Use Monotonic Deque or Hash Map instead [24, 25].

---

## 6. Edge Cases & Debugging Framework

### A. Key Edge Cases
- **Empty or Single-Element Inputs**: Handle empty arrays/strings or inputs smaller than window size $K$ [26].
- **Unreachable Targets**: Return sensible defaults (e.g., `0` or `-1`) when no valid window exists [26].
- **Individual Constraint Violators**: Negative values in positive-sum problems or invalid characters that force `left` to skip past [26, 27].

### B. 4-Step Debugging Protocol
When test cases fail on a sliding window solution, follow this systematic order [29, 30]:
1. **Print Boundaries & State**: Log `left`, `right`, and `running_state` at each iteration to trace expansion/contraction [29].
2. **Test Validity Condition Independently**: Isolate and test the boolean validity check on manual edge cases [29].
3. **Verify Result Update Location**: Ensure result variables are updated strictly when the window is in a valid state [29].
4. **Check Pointer Off-by-One Errors**: Verify loop bounds and ensure no elements are skipped during initialization [30].
