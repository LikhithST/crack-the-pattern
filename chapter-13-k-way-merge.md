# Chapter 13: K-way Merge

## Executive Overview
The **K-way Merge** pattern merges $K$ sorted arrays, lists, or streams into a single unified sorted sequence [200]. While merging 2 sorted sequences uses Parallel Two Pointers in $O(N)$ time [200], naive pairwise merging of $K$ sequences requires $O(N \cdot K)$ comparisons [201].

Using a **Min Heap of size $K$** reduces comparison overhead per step from $O(K)$ to $O(\log K)$, producing an optimal **$O(N \log K)$ time and $O(K)$ space** algorithm (where $N$ is total elements across all sequences) [200, 201].

---

## 1. Core Mechanics & Metadata Tracking

To continue merging, whenever an element is popped from the heap, the algorithm must know **which exact sequence it came from** so it can fetch the next candidate [200, 202].

```
               [ List 0: 1 ➔ 4 ➔ 7 ]
               [ List 1: 2 ➔ 5 ➔ 8 ]
               [ List 2: 3 ➔ 6 ➔ 9 ]
                         │
                         ▼
        ┌──────────────────────────────────┐
        │       MIN HEAP (Size K = 3)      │
        ├──────────────────────────────────┤
        │ Tuples: (val, list_idx, node_ptr)│ [200, 205]
        └────────────────┬─────────────────┘
                         │
        Pop smallest (1) ┴ ➔ Push next from List 0 (4) [201]
```

---

## 2. Universal Algorithm Template (Merge $K$ Sorted Lists)

```python
import heapq

def merge_k_lists(lists):
    min_heap = []
    
    # 1. Initialize heap with first node of each non-empty list [200, 204]
    for i, lst in enumerate(lists):
        if lst:
            # Metadata tuple: (node.val, list_index, node_object) [200, 205]
            heapq.heappush(min_heap, (lst.val, i, lst))
            
    dummy = ListNode(0)
    curr = dummy
    
    # 2. Process heap until exhausted [201]
    while min_heap:
        val, list_idx, node = heapq.heappop(min_heap) # Pop smallest [201]
        curr.next = node
        curr = curr.next
        
        # 3. Fetch next element from SAME source sequence [201, 202]
        if node.next:
            heapq.heappush(min_heap, (node.next.val, list_idx, node.next))
            
    return dummy.next
```

---

## 3. High-Frequency Variations

### Variation 1: Smallest Range Covering Elements from $K$ Lists
- **Problem**: Given $K$ sorted lists, find the smallest range $[a, b]$ that includes at least one number from each of the $K$ lists [201, 204].
- **Algorithm**:
  1. Insert first element of each list into Min Heap, maintaining a running `current_max` across active heap elements [204].
  2. At each step, `current_min = min_heap[0].val` [204].
  3. Evaluate candidate range `[current_min, current_max]` [204]. If `current_max - current_min` is smaller than best range seen so far, record it [204].
  4. Pop `current_min` and push the next element from its source list, updating `current_max = max(current_max, next_val)` [204].
  5. Terminate as soon as any list is exhausted [202, 204].

---

### Variation 2: Find $K$ Pairs with Smallest Sums
- **Problem**: Two sorted arrays `nums1` and `nums2`. Find $K$ pairs $(u, v)$ with smallest sums [206].
- **On-Demand Cartesian Generation**: Do not generate all $N \cdot M$ pairs [206]. Initialize Min Heap with pairs `(nums1[i] + nums2[0], i, 0)` for `i` from $0$ to $\min(N, K)$ [206]. When `(sum, i, j)` is popped, push `(nums1[i] + nums2[j+1], i, j+1)` into heap [206].
- **Complexity**: $O(K \log K)$ time and $O(K)$ space [206].

---

## 4. Edge Cases & Debugging Protocols

### A. High-Frequency Candidate Pitfalls
1. **Losing Source Metadata**: Storing raw values without list indices/pointers makes it impossible to know where to fetch the next element [202, 205].
2. **Tuple Comparison Collisions**: In Python, `(val, list_idx, node)` compares `list_idx` as tie-breaker if `val` matches [203, 205]. If storing custom objects directly without tie-breaker indices, Python raises `TypeError: '<' not supported between instances` [203, 205]. Always include an integer `list_idx` or counter as a secondary tuple element [203, 205].
3. **Empty Input Sequences**: Passing empty lists `[]` inside the input array must be filtered out during heap initialization to prevent null pointer exceptions [204, 205].
