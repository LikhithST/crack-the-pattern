# Chapter 5: Merge Intervals

## Executive Overview
The **Merge Intervals** pattern addresses problems revolving around overlapping ranges, scheduling conflicts, and resource allocations [54]. The foundational insight of this pattern is that **intervals sorted by start time** allow all temporal relationships (overlaps, gaps, containment) to be determined in a single linear pass [54].

Sorting the intervals is a **non-negotiable prerequisite**; without sorting, comparing every interval against every other interval requires $O(N^2)$ time [54]. With sorting, the total complexity is **$O(N \log N)$ time and $O(N)$ space** [54].

---

## 1. Overlap Conditions & Mathematical Unification

Two sorted intervals $A = [start_A, end_A]$ and $B = [start_B, end_B]$ (where $start_A \le start_B$) can relate in one of three ways:

```
1. NO OVERLAP:           [ A ]
                                 [ B ]          Condition: B.start > A.end

2. PARTIAL OVERLAP:      [    A    ]
                              [    B    ]       Condition: B.start <= A.end
                                                Merged: [A.start, max(A.end, B.end)]

3. COMPLETE CONTAINMENT: [       A       ]
                            [   B   ]           Condition: B.start <= A.end
                                                Merged: [A.start, max(A.end, B.end)]
```

### The Unification Trick
Notice that for both Partial Overlap and Complete Containment, the merged interval end is given by **`max(A.end, B.end)`** [56]. You do **not** need separate special-case logic for complete containment; taking the maximum end value mathematically covers both cases cleanly [56].

---

## 2. Universal Algorithm Template & Variations

### Variation 1: Merge Overlapping Intervals (Standard Template)
1. Sort intervals by start time in ascending order [54].
2. Initialize output list with the first interval [54].
3. Iterate through remaining intervals:
   - If `current.start <= last_merged.end`: Overlap exists. Update `last_merged.end = max(last_merged.end, current.end)` [54, 56].
   - Else: No overlap. Append `current` to output list [54].

```python
def merge_intervals(intervals):
    if not intervals:
        return []
        
    # 1. Non-negotiable sorting by start time
    intervals.sort(key=lambda x: x[0])
    
    merged = [intervals[0]]
    
    for current in intervals[1:]:
        last_start, last_end = merged[-1]
        curr_start, curr_end = current
        
        if curr_start <= last_end: # Overlap detected [54]
            merged[-1][1] = max(last_end, curr_end) # Unification math [56]
        else:
            merged.append(current) # Gap detected [54]
            
    return merged
```

---

### Variation 2: Insert Interval
- **Objective**: Insert a new interval into an already sorted list of non-overlapping intervals, merging if necessary [55, 57].
- **Optimized Approach ($O(N)$ time)**:
  1. **Phase 1**: Add all existing intervals that end before `newInterval.start` (`curr.end < newInterval.start`) [58].
  2. **Phase 2**: Merge all intervals that overlap with `newInterval` (`curr.start <= newInterval.end`) by expanding `newInterval`:
     - `newInterval.start = min(newInterval.start, curr.start)`
     - `newInterval.end = max(newInterval.end, curr.end)`
  3. **Phase 3**: Add the expanded `newInterval`, then append all remaining intervals [58].

---

### Variation 3: Interval Intersections
- **Objective**: Find all overlapping regions between two separate sorted interval lists $A$ and $B$ [55].
- **Mechanics**: Parallel Two Pointers (`ptrA`, `ptrB`) [55].
  - Overlap region: `[max(A[i].start, B[j].start), min(A[i].end, B[j].end)]` [55].
  - Advance pointer pointing to the interval that **ends first** (`min(A[i].end, B[j].end)`), as it cannot overlap with any subsequent intervals [55].

---

### Variation 4: Gap Finding (Employee Free Time)
- **Objective**: Find mutual non-overlapping gaps across multiple interval lists [55, 58].
- **Mechanics**: Merge all schedule intervals together into a single unified list. The gaps between consecutive merged intervals (`gap = [merged[i].end, merged[i+1].start]`) represent common free time [55, 58].

---

### Variation 5: Meeting Rooms II (Minimum Resources / Peak Overlap)
- **Objective**: Find the minimum number of meeting rooms required to host all meetings [55, 56].
- **Distinct Mechanics**: This does **not** merge intervals. Instead, it measures **maximum concurrent overlap** at any single point in time [55].
- **Two-Pointer Chronological Sweep Algorithm**:
  1. Separate all start times and end times into two independent arrays [55].
  2. Sort both arrays independently in ascending order [55].
  3. Use parallel pointers (`start_ptr`, `end_ptr`):
     - If `start_times[start_ptr] < end_times[end_ptr]`: A meeting started before the earliest ending meeting finished $\rightarrow$ Increment room count, `start_ptr++` [55].
     - Else: A meeting finished $\rightarrow$ Decrement room count, `end_ptr++` [55].

---

## 3. Boundary Nuances & Edge Cases

### A. Boundary Touches (`A.end == B.start`)
- **Standard Merge Problem**: Intervals touching at a single point (e.g., `[1, 3]` and `[3, 5]`) **are merged** into `[1, 5]` because they represent continuous coverage (`curr.start <= last_end`) [57].
- **Meeting Rooms Problem**: Meetings touching at a boundary (Meeting 1 ends at 3, Meeting 2 starts at 3) **do NOT conflict** because room room transition is instantaneous (`start < end` instead of `<=`) [57].
- **Always clarify boundary inclusion (`<=` vs `<`) with your interviewer upfront** [57].

### B. High-Frequency Edge Cases
- **Empty or Single Interval List**: Return input immediately [56].
- **Total Containment**: Small interval completely inside a huge interval (e.g., `[1, 10]` and `[2, 3]`). Covered automatically by `max(A.end, B.end)` [56].
- **Unsorted Input**: Sorting step omitted is the #1 candidate failure point [54, 56].
