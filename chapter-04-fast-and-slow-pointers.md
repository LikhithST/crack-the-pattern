# Chapter 4: Fast and Slow Pointers

## Executive Overview
The **Fast and Slow Pointers** pattern (also known as **Floyd’s Tortoise and Hare algorithm**) uses two pointers that traverse a sequence or pointer-based data structure at different speeds [44]. While standard cycle detection using a Hash Set requires $O(N)$ extra memory, Fast and Slow Pointers solves cycle detection and sequence mid-point problems in **$O(N)$ time and $O(1)$ auxiliary space** [44, 47].

---

## 1. Core Mechanics & Mathematical Proof

### A. Traversal Ratio
- **Slow Pointer (`slow`)**: Advances **1 step** per iteration [44].
- **Fast Pointer (`fast`)**: Advances **2 steps** per iteration [44].

### B. Mathematical Guarantee of Convergence
When entering a cycle of length $C$:
1. With each iteration, `fast` gains exactly $1$ step on `slow` ($2 - 1 = 1$) [45, 48].
2. The relative distance between `fast` and `slow` inside the cycle decreases by $1$ on every step [48, 49].
3. Therefore, `fast` is mathematically guaranteed to catch up to and meet `slow` within at most $C$ iterations after `slow` enters the cycle [45, 49]. It will never skip past `slow` because pointer positions change in discrete steps of $1$ unit relative distance [49].

```
NON-CYCLE:  head ➔ [1] ➔ [2] ➔ [3] ➔ [4] ➔ null (fast reaches null ➔ NO CYCLE) [44]

CYCLE:      head ➔ [1] ➔ [2] ➔ [3] ➔ [4]
                          ▲          │
                          └──────────┘ (fast meets slow ➔ CYCLE DETECTED) [44, 45]
```

---

## 2. The Three Fundamental Variations

### Variation 1: Cycle Detection (Boolean Check)
- **Objective**: Determine whether a linked list or pointer sequence contains a loop [44].
- **Termination**: If `fast` or `fast.next` becomes `null`, no cycle exists [44]. If `slow == fast`, a cycle exists [45].

```python
def has_cycle(head):
    if not head or not head.next:
        return False
        
    slow = head
    fast = head.next  # Initialized offset to avoid false positive on loop start [50]
    
    while fast and fast.next:
        if slow == fast:
            return True
        slow = slow.next
        fast = fast.next.next
        
    return False
```

### Variation 2: Finding Cycle Start Node
- **Objective**: Identify the exact node where a cycle begins [45].
- **Algorithm (2-Phase Process)** [45]:
  1. **Phase 1**: Run standard cycle detection until `slow` and `fast` meet at node $M$ inside the cycle [45].
  2. **Phase 2**: Reset `slow` back to `head` [45]. Keep `fast` at meeting point $M$.
  3. **Phase 3**: Move both `slow` and `fast` at the **same speed (1 step at a time)** [45]. The node where they meet again is the exact **start of the cycle** [45].

### Variation 3: Finding Middle of Linked List
- **Objective**: Locate the middle node of a linked list in a single pass [46].
- **Mechanics**: Move `slow` by 1 step and `fast` by 2 steps [46]. When `fast` reaches the end (`null` or last node), `slow` will be positioned exactly at the midpoint [46].
- **Even-Length Decision**: Depending on requirements, returning the first middle vs. second middle is controlled by whether `fast` stops at `fast.next == null` vs `fast == null` [47].

---

## 3. Advanced Applications & Cross-Domain Extensions

### A. Virtual Linked Lists in Arrays (Find Duplicate Number)
- **Domain**: Arrays where each element's value represents the index of the next element (`next = arr[i]`) [46].
- **Insight**: If an array of size $N+1$ contains numbers from $1$ to $N$, at least one duplicate exists [46]. The duplicate value creates multiple incoming edges to the same index, forming a **virtual linked list cycle** [46]. Fast & Slow pointers can locate the duplicate in $O(N)$ time and $O(1)$ space without modifying the array [46].

### B. Implicit Sequence Cycles (Happy Number)
- **Domain**: Transforming a number by repeatedly replacing it with the sum of the squares of its digits [49].
- **Insight**: The process either reaches $1$ or enters a repeating number cycle that never reaches $1$ [49]. By treating each generated number as a node in a virtual sequence, Fast & Slow pointers detect if the process is trapped in a non-1 cycle [49].

---

## 4. Pattern Combination: Multi-Stage Linked List Problems

Complex interview questions often require chaining Fast & Slow Pointers with other patterns [52, 53]:

| Problem | Stage 1 (Fast & Slow) | Stage 2 (Linked List Reversal) | Stage 3 (Two Pointers) |
| :--- | :--- | :--- | :--- |
| **Reorder List** | Find list midpoint [52, 53] | Reverse second half of list [53] | Interleave first and second halves [53] |
| **Palindrome List** | Find list midpoint [53] | Reverse second half of list [53] | Compare values from head and reversed tail [53] |

---

## 5. Edge Cases & Debugging Protocols

### A. Critical Edge Cases
- **Empty List or Single Node**: `head is None` or `head.next is None` must be checked upfront to avoid null pointer exceptions [47].
- **Pointer Initialization Bug**: If `slow` and `fast` are both initialized to `head` and evaluated in a `while slow != fast:` loop, the condition fails immediately on iteration 1 [50]. Fix by setting `fast = head.next` or using a move-first loop structure [50].
- **Even vs. Odd List Termination**: Ensure `fast` checks both `fast` AND `fast.next` before calling `fast.next.next` to avoid crash at end of list [47, 50].

### B. Interview Strategy: Hash Set vs. Fast & Slow
- If interviewer explicitly mandates **$O(1)$ extra space**: Implement Fast & Slow Pointers directly [47, 52].
- If space is unconstrained: Mention that a Hash Set is simpler to explain, then offer to optimize to Fast & Slow Pointers for $O(1)$ space [47, 52].
