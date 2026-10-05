# Chapter 12: Top K Elements

## Executive Overview
The **Top K Elements** pattern finds the $K$ largest, $K$ smallest, $K$ most frequent, or $K$ closest elements in a dataset [190]. The fundamental insight is that **you do not need to sort the full dataset ($O(N \log N)$)** [190]. By filtering elements through a Priority Queue (Heap) capped at size $K$, the runtime drops to **$O(N \log K)$ time and $O(K)$ auxiliary space** [190, 192].

When $K \ll N$, $O(N \log K)$ is substantially faster than full sorting [190, 192].

---

## 1. The Counter-Intuitive Heap Inversion Rule

The #1 source of candidate confusion and bug creation is selecting the wrong heap type [191, 193, 197]:

```
1. FIND TOP 'K' LARGEST ELEMENTS  ➔  USE A MIN HEAP OF SIZE 'K' [190, 191]
   • Why? The top of a Min Heap holds the SMALLEST of the top K elements seen so far [191].
   • When heap size exceeds K, popping removes the overall smallest element,
     preserving the K largest candidates [190, 191].

2. FIND TOP 'K' SMALLEST ELEMENTS ➔  USE A MAX HEAP OF SIZE 'K' [191]
   • Why? The top of a Max Heap holds the LARGEST of the bottom K elements seen so far [191].
   • When heap size exceeds K, popping removes the overall largest element,
     preserving the K smallest candidates [191].
```

---

## 2. Universal Algorithm Template & Variations

### Variation 1: Top $K$ Largest Elements Template

```python
import heapq

def find_k_largest(nums, k):
    # Maintain a MIN HEAP of size K [190, 191]
    min_heap = []
    
    for num in nums:
        heapq.heappush(min_heap, num)
        
        # Enforce heap size limit of K [190, 197]
        if len(min_heap) > k:
            heapq.heappop(min_heap)  # Discard the smallest element [190, 191]
            
    return list(min_heap)  # Heap contains exactly the K largest elements [191]
```

---

### Variation 2: Top $K$ Most Frequent Elements
1. Build a Frequency Hash Map (`frequency_map[val] += 1`) in $O(N)$ time [191].
2. Iterate over `(val, freq)` pairs in the hash map [191].
3. Push `(freq, val)` tuples into a Min Heap of size $K$ [191]. Python heaps compare tuples by their first element (`freq`) automatically [191].

```python
import heapq
from collections import Counter

def top_k_frequent(nums, k):
    count = Counter(nums) # O(N) frequency map [191]
    min_heap = []
    
    for val, freq in count.items():
        heapq.heappush(min_heap, (freq, val)) # Heap ordered by frequency [191]
        if len(min_heap) > k:
            heapq.heappop(min_heap) # Discard lowest frequency [191]
            
    return [val for freq, val in min_heap]
```

---

### Variation 3: $K$ Closest Points to Origin
- **Distance Metric**: Compute Euclidean distance squared $X^2 + Y^2$ [192].
- **Algorithm**: Maintain a **Max Heap** of size $K$ storing `(-distance, point)` [191, 192]. Negating distance allows Python's default min-heap to function as a Max Heap, popping distant points when size exceeds $K$ [191, 199].

---

## 3. Algorithm Comparison Matrix: Heap vs. QuickSelect vs. Sorting

| Metric / Scenario | Heap Filtering | QuickSelect Algorithm | Full Sorting |
| :--- | :--- | :--- | :--- |
| **Time Complexity** | $O(N \log K)$ [190, 192] | $O(N)$ average / $O(N^2)$ worst [192, 194] | $O(N \log N)$ [190, 192] |
| **Space Complexity** | $O(K)$ auxiliary space [192] | $O(1)$ in-place space [192, 194] | $O(1)$ or $O(N)$ [192] |
| **Streaming Data Support**| **Yes** (processes items 1-by-1) [192, 193] | **No** (requires full array in memory) [193] | **No** (requires re-sorting) [193] |
| **Input Mutability** | Preserves input array [192] | Mutates input array in-place [192, 194] | Mutates or copies [192] |

---

## 4. Edge Cases & Implementation Traps

### A. High-Frequency Candidate Mistakes
1. **Using Max Heap for Top $K$ Largest**: If you push all elements into a Max Heap without enforcing size $K$, the heap grows to size $N$ and never discards elements [193, 197]. Popping $K$ times takes $O(N \log N)$ time, completely destroying the $O(N \log K)$ performance gain [193, 197].
2. **Forgetting Size Enforcement**: Forgetting `if len(heap) > k: heapq.heappop(heap)` lets the heap expand to $O(N)$ space [197].
3. **Tie-Breaking Logic**: When two elements share identical frequency or distance, default tuple comparison might compare second elements [196]. In languages without custom comparators, define explicit tuple structure `(primary_key, secondary_key, val)` [196, 199].
