# Chapter 9: Two Heaps

## Executive Overview
The **Two Heaps** pattern is a specialized algorithm for tracking order statistics—most notably the **running median**—over continuously arriving data streams or dynamic data windows [166]. By maintaining two heaps with opposite comparison orderings, the median can be calculated in **$O(1)$ constant time** following each $O(\log N)$ element insertion [167, 168].

---

## 1. Core Mechanics & Size Invariants

```
                DATA STREAM
                    │
                    ▼
          ┌───────────────────┐
          │  New Element  num │
          └─────────┬─────────┘
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
┌───────────────────┐ ┌───────────────────┐
│     MAX HEAP      │ │     MIN HEAP      │
│  (Smaller Half)   │ │   (Larger Half)   │
├───────────────────┤ ├───────────────────┤
│ Stores values <=  │ │ Stores values >   │
│ median [166]      │ │ median [166, 167] │
│ Top = Largest of  │ │ Top = Smallest of │
│ smaller half [166]│ │ larger half [166] │
└───────────────────┘ └───────────────────┘
```

### The Size Invariant
To ensure the median is always at the top of one or both heaps:
1. **Size Difference Constraint**: `len(max_heap)` and `len(min_heap)` must differ by **at most 1 element** (`abs(len(max_heap) - len(min_heap)) <= 1`) [166, 167].
2. **Convention**: By convention, `max_heap` is allowed to hold 1 extra element during odd-numbered total elements [167].

---

## 2. Universal 3-Step Algorithm

### Step 1: Insertion Decision
When a new element `num` arrives [167]:
- If `max_heap` is empty or `num <= -max_heap[0]` (in Python min-heap representation), push `num` to **`max_heap`** [167].
- Otherwise, push `num` to **`min_heap`** [167].

### Step 2: Rebalancing
Maintain size balance [167]:
- If `len(max_heap) > len(min_heap) + 1`: Pop top of `max_heap` and push to `min_heap` [167].
- If `len(min_heap) > len(max_heap)`: Pop top of `min_heap` and push to `max_heap` [167].

### Step 3: Median Computation ($O(1)$ Time)
- **Odd total elements**: Median is `-max_heap[0]` [167].
- **Even total elements**: Median is `(-max_heap[0] + min_heap[0]) / 2.0` [167, 168].

```python
import heapq

class MedianFinder:
    def __init__(self):
        # max_heap stores smaller half (negated values for Python min-heap) [166, 167]
        self.max_heap = [] 
        # min_heap stores larger half [166]
        self.min_heap = [] 

    def addNum(self, num: int) -> None:
        # 1. Insertion [167]
        if not self.max_heap or num <= -self.max_heap[0]:
            heapq.heappush(self.max_heap, -num)
        else:
            heapq.heappush(self.min_heap, num)
            
        # 2. Rebalance [167]
        if len(self.max_heap) > len(self.min_heap) + 1:
            heapq.heappush(self.min_heap, -heapq.heappop(self.max_heap))
        elif len(self.min_heap) > len(self.max_heap):
            heapq.heappush(self.max_heap, -heapq.heappop(self.min_heap))

    def findMedian(self) -> float:
        # 3. $O(1)$ Median Calculation [167, 168]
        if len(self.max_heap) > len(self.min_heap):
            return float(-self.max_heap[0])
        return (-self.max_heap[0] + self.min_heap[0]) / 2.0
```

---

## 3. Advanced Variation: Sliding Window Median & Lazy Deletion

When computing a running median inside a sliding window of size $K$, elements leave the window from the left [168, 170]. Standard heaps do not support $O(\log N)$ arbitrary element removal [168, 170].

### The Lazy Deletion Technique
Rather than searching and rebuilding the heap, record removed elements in a **Hash Map of invalidated counts** (`invalidated_map[num] += 1`) [170, 171]:
1. Mark the leaving element as invalidated in the hash map [170].
2. Adjust size counts of valid elements in both heaps [171].
3. **Lazy Cleanup**: Whenever `max_heap` or `min_heap` top contains an invalidated element, pop and discard it before performing median lookup [170, 171].
4. **Complexity**: Maintains $O(\log K)$ amortized insertion/deletion per step [171, 173].

---

## 4. Extension: Percentile Tracking
The Two Heaps pattern can be generalized beyond medians (50th percentile) to track **any arbitrary percentile** (e.g., 90th percentile SLA response time) [173]:
- Adjust the heap size ratio constraint [173].
- For 90th percentile: Maintain `len(max_heap) : len(min_heap) == 90% : 10%` [173]. Top of `min_heap` provides the 90th percentile value in $O(1)$ time [173].

---

## 5. When NOT to Use Two Heaps
1. **Static Dataset**: If the full array is given upfront and doesn't stream, simply sort once ($O(N \log N)$) or use QuickSelect ($O(N)$ average) [169].
2. **Strict $O(1)$ Space Required**: Two Heaps stores all stream elements, taking $O(N)$ space [169].
