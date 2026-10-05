# Chapter 7: Linked List Reversal

## Executive Overview
The **In-place Reversal of a Linked List** pattern modifies pointer directions of a linked list without allocating extra nodes, achieving **$O(N)$ time and $O(1)$ auxiliary space** [63]. While conceptually straightforward, pointer manipulation is notoriously error-prone in interview settings [63].

Mastering this pattern requires tracking three pointers simultaneously (`prev`, `curr`, `next`) and maintaining strict reference safety so that the remaining unreversed list is never lost [63].

---

## 1. Core Mechanics & The 3-Pointer Loop

### The Pointer Cascade
At every step of iteration, before modifying `curr.next`, you **must save `curr.next`** into `next_node` [63]. Overwriting `curr.next` without saving severs access to the remainder of the list [63, 65].

```
Initial State:
[ prev = None ]     [ curr = Head ] ➔ [ Node 2 ] ➔ [ Node 3 ]

Step-by-Step State Transition:
1. Save next reference:  next_node = curr.next           [63]
2. Reverse link:         curr.next = prev                [63]
3. Shift prev forward:   prev = curr                     [63]
4. Shift curr forward:   curr = next_node                [63]
```

```python
def reverse_full_list(head):
    prev = None
    curr = head
    
    while curr:
        next_node = curr.next  # 1. Save reference [63]
        curr.next = prev       # 2. Reverse pointer [63]
        prev = curr            # 3. Advance prev [63]
        curr = next_node       # 4. Advance curr [63]
        
    return prev  # 'prev' is the new head of the reversed list [63]
```

---

## 2. Variations & Sublist Reconnection Mechanics

### Variation 1: Reverse Sublist (Between Position $L$ and $R$)
- **Challenge**: Reversing nodes from index $L$ to $R$ while keeping surrounding nodes intact [64].
- **Reconnection Trick**: You must maintain references to the node immediately before position $L$ (`left_prev`) and the original node at position $L$ (`sublist_head`, which becomes the sublist tail after reversal) [64, 65].

```python
def reverse_between(head, left, right):
    if not head or left == right:
        return head
        
    # Use dummy node to elegantly handle edge case where left == 1 (head changes) [65]
    dummy = ListNode(0)
    dummy.next = head
    left_prev = dummy
    
    # 1. Reach node prior to 'left' position [64]
    for _ in range(left - 1):
        left_prev = left_prev.next
        
    # 2. Reverse sublist between left and right [64]
    curr = left_prev.next
    prev = None
    for _ in range(right - left + 1):
        next_node = curr.next
        curr.next = prev
        prev = curr
        curr = next_node
        
    # 3. Reconnect reversed sublist [64, 65]
    # 'left_prev.next' was original sublist head; now it's sublist tail
    left_prev.next.next = curr  # Connect sublist tail to remaining list [64, 65]
    left_prev.next = prev       # Connect left_prev to new sublist head [64, 65]
    
    return dummy.next
```

---

### Variation 2: Reverse Nodes in $K$-Group
- **Problem**: Reverse nodes in groups of $K$ at a time. If the number of remaining nodes is less than $K$, leave them as is [64, 65].
- **Algorithm**:
  1. Count $K$ nodes ahead to verify if a full group of size $K$ exists [65].
  2. If a full group exists, reverse those $K$ nodes using the 3-pointer loop [64].
  3. Recursively or iteratively connect the tail of the newly reversed $K$-group to the result of processing the remaining list [64, 65].

---

## 3. Edge Cases & Debugging Protocols

### A. Critical Failure Points
1. **Dangling Head Pointer / Null Tail**: Failing to initialize `prev = None` at the start causes the old head's `next` pointer to remain pointing to its old neighbor, creating a cycle or losing list termination [63, 65].
2. **Sublist Starts at Position 1 ($L = 1$)**: Reversing from index 1 changes the list's `head` node [65]. **Always use a `dummy` node** pointing to `head` (`dummy.next = head`) to handle $L=1$ uniformly without special conditional code [65].
3. **Reference Loss**: Forgetting to store `curr.next` before reassigning pointers destroys list connectivity [63, 65].

### B. 3-Step Reversal Verification Checklist
1. **Dummy Node**: Did you initialize a `dummy` node to safeguard `head` changes? [65]
2. **Reconnection Verification**: Did you update BOTH `left_prev.next` AND `sublist_tail.next` after sublist reversal? [64, 65]
3. **Loop Boundaries**: Is the sublist reversal loop executing strictly $R - L + 1$ times? [64]
