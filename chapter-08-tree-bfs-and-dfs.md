# Chapter 8: Tree BFS and DFS

## Executive Overview
Tree traversal algorithms are the gateway to solving hierarchical data structure challenges [157]. **Breadth-First Search (BFS)** and **Depth-First Search (DFS)** are fundamental strategies that reveal distinct structural properties of trees [157]. Selecting the appropriate traversal is the most critical strategic decision in any tree problem [157].

---

## 1. Core Mechanics & Comparison

| Characteristic | Tree BFS (Level-Order) | Tree DFS (Depth-First) |
| :--- | :--- | :--- |
| **Primary Data Structure** | Queue (FIFO) [157] | Call Stack (Recursion) or Stack (LIFO) [158, 159] |
| **Exploration Strategy** | Processes nodes level by level across depth [157] | Explores along branches as deep as possible first [158] |
| **Shortest Path Guarantee** | **Guaranteed** shortest path (minimum edges) [158, 160] | No path length guarantees [160] |
| **Auxiliary Space** | $O(W)$ where $W$ is maximum width of tree ($O(N)$ worst) [162] | $O(H)$ where $H$ is height ($O(\log N)$ balanced, $O(N)$ skewed) [159, 162] |
| **Best Choice When** | Tree is wide, or solution is near root / level-based [158, 160] | Tree is deep, or solution is near leaf / path-based [160] |

---

## 2. Tree BFS: Level-By-Level Processing

### Level-Order Snapshot Template
The defining characteristic of Tree BFS is taking a **snapshot of the queue size (`level_size`)** at the start of each outer loop iteration [158]. This guarantees that the inner loop processes exactly one complete horizontal level of the tree [158].

```python
from collections import deque

def tree_bfs_level_order(root):
    if not root:
        return []
        
    result = []
    queue = deque([root])  # FIFO queue initialized with root [157]
    
    while queue:
        level_size = len(queue)  # Snapshot current level node count [158]
        current_level = []
        
        for _ in range(level_size): # Process strictly one level [158]
            node = queue.popleft()
            current_level.append(node.val)
            
            # Enqueue children for next level [157]
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
                
        result.append(current_level)
        
    return result
```

### Key BFS Variations
1. **Zigzag / Spiral Level Order**: Use a boolean direction flag or `deque`. Reverse the level list insertion order on alternating levels [163, 164].
2. **Right Side View of Tree**: Process level order and capture the last node (`i == level_size - 1`) of each level [158].
3. **Connect Level Order Siblings**: Connect `node.next` to `queue[0]` for all nodes except the last in the level [158].

---

## 3. Tree DFS: Traversals & Traversal Orders

Depth-First Search explores branches recursively or iteratively using a stack [158, 159]. The choice of **traversal order** determines when parent nodes are evaluated relative to their children [159]:

```
         (Root)
          /  \
      (Left) (Right)
```

### A. Preorder Traversal (Root $\rightarrow$ Left $\rightarrow$ Right)
- **Mechanics**: Process parent node **before** traversing children [159].
- **Primary Applications**:
  - Tree cloning / copying [159].
  - Tree serialization / deserialization (encoding tree structure into a string) [160].
  - Root-to-leaf path sum calculations [160].

### B. Inorder Traversal (Left $\rightarrow$ Root $\rightarrow$ Right)
- **Mechanics**: Process left child, then parent node, then right child [159].
- **Binary Search Tree (BST) Invariant**: Inorder traversal of a BST visits node values in strictly **increasing sorted order** [159, 164].
- **Primary Applications**:
  - BST Validation: Verify if visited values are strictly monotonically increasing [164].
  - Find $K$-th Smallest Element in BST: Stop traversal after visiting $K$ elements [164].

### C. Postorder Traversal (Left $\rightarrow$ Right $\rightarrow$ Root)
- **Mechanics**: Process both children **before** evaluating the parent node [159].
- **Bottom-Up Property**: Parent receives structural information evaluated from both subtrees [159, 160].
- **Primary Applications**:
  - Calculating tree height / depth [159].
  - Checking tree balance (e.g., AVL property) [159].
  - **Lowest Common Ancestor (LCA)**: Propagate non-null node matches upward from children [160].

---

## 4. Iterative DFS vs. Recursive DFS

While recursive DFS is elegant, iterative DFS using an explicit stack avoids **recursion call stack overflow** on deep trees [159].

```python
# Iterative Preorder DFS (Stack LIFO)
def iterative_preorder(root):
    if not root:
        return []
    result, stack = [], [root]
    
    while stack:
        node = stack.pop()
        result.append(node.val)
        
        # PUSH RIGHT FIRST so LEFT is popped and processed first! [163]
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
            
    return result
```

---

## 5. Edge Cases & Debugging Protocols

### A. High-Frequency Traversal Traps
1. **Adding Null Children to Queue/Stack**: In iterative traversals, failing to check `if node.left:` before enqueuing adds `None` references, causing `AttributeError: 'NoneType' object has no attribute 'val'` when popped [161, 162].
2. **Missing Recursion Base Case**: In recursive DFS, failing to check `if not node: return` on the first line causes infinite recursion crashes [161].
3. **Confusing Space Complexity**: Assuming DFS is always $O(1)$ space. Recursive DFS uses $O(H)$ stack memory, which equals $O(N)$ for skewed trees [159, 162].

### B. Decision Matrix for Interview Strategy
- Asking for **shortest path** or **minimum depth**? $\rightarrow$ Use **BFS** [158, 160].
- Problem involves **Binary Search Tree** properties? $\rightarrow$ Use **Inorder DFS** [159, 164].
- Problem requires **combining subtree results** at parent? $\rightarrow$ Use **Postorder DFS** [159, 160].
- Problem involves **root-to-leaf paths**? $\rightarrow$ Use **Preorder DFS** with a running path state [160].
