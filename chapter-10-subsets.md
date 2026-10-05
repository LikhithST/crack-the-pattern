# Chapter 10: Subsets (Combinatorial Backtracking)

## Executive Overview
The **Subsets** (or Combinatorial Backtracking) pattern addresses problems requiring the generation of all possible combinations, permutations, or subsets of a dataset [174]. Unlike optimization patterns that reduce time complexity, Subsets generates an **exponential output space** ($O(2^N)$ for subsets, $O(N!)$ for permutations) [172, 174].

The primary engineering challenge is ensuring **correctness, completeness, and efficient branch pruning** to eliminate invalid search paths as early as possible [174, 178].

---

## 1. Two Fundamental Approaches

```
                      ROOT [ ]
                    /          \
            INCLUDE [1]        EXCLUDE [ ]
           /          \          /        \
      INC [1,2]    EXC [1]   INC [2]     EXC [ ]
```

### Approach 1: Iterative BFS Generation
- **Mechanics**: Start with a result list containing only the empty set `[[]]` [174]. For each element in the input, iterate through all existing subsets in the result list and create new subsets by appending the current element [174].
- **Output Size**: Generates exactly $2^N$ subsets for an input of size $N$ [172, 174].

```python
def subsets_iterative(nums):
    result = [[]] # Initial seed value [174]
    
    for num in nums: # Iterate each input element [174]
        # Append num to a copy of all existing subsets [174]
        result += [curr + [num] for curr in result]
        
    return result
```

---

### Approach 2: Recursive Backtracking (Preferred for Interviews)
- **Mechanics**: Constructs a **decision tree** using recursion [175]. Maintains a single mutable `path` list and an `index` pointer [175].
- **The Backtracking Step**: Modify `path` in-place (`path.append()`), recurse, and then revert the state (`path.pop()`) upon return [176, 179]. Reusing a single path list reduces memory overhead from exponential $O(2^N)$ to linear $O(N)$ call stack space [179].

```python
def subsets_backtracking(nums):
    result = []
    
    def backtrack(start_idx, current_path):
        # Every state in the decision tree is a valid subset [174]
        result.append(list(current_path))
        
        for i in range(start_idx, len(nums)):
            # 1. Choose: Include element [175]
            current_path.append(nums[i])
            
            # 2. Explore: Recurse to next index [175]
            backtrack(i + 1, current_path)
            
            # 3. Un-choose (Backtrack step): Revert state [176, 179]
            current_path.pop()
            
    backtrack(0, [])
    return result
```

---

## 2. Deduplication with Duplicate Inputs

When the input array contains duplicate elements (e.g., `[1, 2, 2]`), naive subset generation produces duplicate subsets [174].

### The Non-Negotiable Deduplication Protocol
1. **Sort the input array first** to group identical numbers adjacently [174, 179].
2. **Skip Duplicate Branches**: Inside the loop, if the current element matches the previous element AND `i > start_idx`, skip processing [176, 178].

```python
def subsets_with_dup(nums):
    nums.sort()  # 1. Non-negotiable sorting step [174, 179]
    result = []
    
    def backtrack(start_idx, path):
        result.append(list(path))
        
        for i in range(start_idx, len(nums)):
            # 2. Skip duplicates at the same tree depth level [176, 178]
            if i > start_idx and nums[i] == nums[i - 1]:
                continue
                
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()
            
    backtrack(0, [])
    return result
```

---

## 3. High-Frequency Variations

### Variation A: Combinations (Fixed Size $K$)
- **Condition**: Only record `path` when `len(path) == k` [173, 176].
- **Pruning Optimization**: Stop recursing if the remaining available elements are fewer than required to reach size $K$ (`len(nums) - i < k - len(path)`) [178].

### Variation B: Permutations ($N!$ Output)
- **Condition**: Order matters (`[1, 2]` $\neq$ `[2, 1]`). Every element must appear exactly once [173, 177].
- **Mechanics**: Instead of `start_idx`, iterate `i` from $0$ to $N-1$ at every recursive step, maintaining a `visited` boolean array or swapping elements [177].

---

## 4. When to Use vs. When NOT to Use

### A. Primary Triggers
- Prompt explicitly demands **all valid configurations**, **all combinations**, **all permutations**, or the **power set** [172, 174, 176].
- Constraint scale $N \le 20$ (indicating $O(2^N)$ runtime is expected and acceptable) [174].

### B. Crucial Counter-Indicators (The DP / Greedy Trap)
- **Existence Query**: If the problem asks whether *any* subset sums to target, **DO NOT generate all subsets** [173, 182]. Use **Dynamic Programming (0/1 Knapsack)** in pseudo-polynomial time [173, 182].
- **Counting Query**: If the problem asks for the *count* of qualifying subsets, use **Dynamic Programming** instead [182].
