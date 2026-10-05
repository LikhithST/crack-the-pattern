# Chapter 14: Topological Sort

## Executive Overview
**Topological Sort** provides a linear ordering of vertices in a **Directed Acyclic Graph (DAG)** such that for every directed edge $A \rightarrow B$, vertex $A$ appears before vertex $B$ [208]. It is the premier pattern for dependency resolution, task scheduling, course prerequisite verification, and build systems (e.g., Bazel, Make, npm) [208, 214].

Kahn's Algorithm (BFS-based) solves Topological Sort in **$O(V + E)$ time and $O(V + E)$ space**, where $V$ is vertices and $E$ is edges [208, 209].

---

## 1. Core Mechanics: Kahn’s BFS Algorithm

```
Step 1: Build Adjacency List & Compute In-Degrees [208]
        In-Degree = Number of incoming directed edges pointing to a vertex [208].

Step 2: Seed Queue with Zero In-Degree Vertices [209]
        Vertices with In-Degree = 0 have NO dependencies and can be processed immediately [209].

Step 3: Process Queue & Decrement Neighbor In-Degrees [209, 211]
        Dequeue vertex 'u' ➔ Add 'u' to Topological Order.
        For each neighbor 'v' of 'u': Decrement in_degree[v] by 1 [211].
        If in_degree[v] reaches 0 ➔ Enqueue 'v' [209].

Step 4: Cycle Validation [209, 210]
        If len(topological_order) == V ➔ Valid DAG Order [210].
        If len(topological_order) < V  ➔ CYCLE DETECTED! (No valid order) [209, 210].
```

---

## 2. Universal Code Template (Course Schedule II)

```python
from collections import deque, defaultdict

def find_order(num_courses, prerequisites):
    # 1. Build Adjacency List and In-Degree Map [208]
    adj_list = defaultdict(list)
    in_degree = {i: 0 for i in range(num_courses)}
    
    for dest, src in prerequisites: # Edge src -> dest [213]
        adj_list[src].append(dest)
        in_degree[dest] += 1
        
    # 2. Seed Queue with vertices having In-Degree == 0 [209]
    queue = deque([v for v in in_degree if in_degree[v] == 0])
    topo_order = []
    
    # 3. Process BFS [209]
    while queue:
        u = queue.popleft()
        topo_order.append(u)
        
        for v in adj_list[u]:
            in_degree[v] -= 1  # Decrement neighbor in-degree [211]
            if in_degree[v] == 0:
                queue.append(v)
                
    # 4. Cycle Check: If total processed < num_courses, a cycle exists [209, 210]
    if len(topo_order) == num_courses:
        return topo_order
    return []  # Return empty array if cycle present [210]
```

---

## 3. High-Frequency Variations

### Variation 1: Alien Dictionary (Hard)
- **Problem**: Derive letter ordering from a sorted dictionary of words written in an alien language [210, 213].
- **Graph Construction**:
  1. Compare adjacent words $W_1$ and $W_2$ character by character [213, 214].
  2. Find the **first differing position**: $W_1[k] \neq W_2[k]$ [214].
  3. Add directed edge $W_1[k] \rightarrow W_2[k]$ (character $W_1[k]$ comes before $W_2[k]$) [214, 215].
  4. **Prefix Rule Edge Case**: If $W_1$ is longer than $W_2$ and $W_2$ is a prefix of $W_1$ (e.g., `"abc"` before `"ab"`), the dictionary is invalid $\rightarrow$ return `""` [215].
- **Execution**: Apply Kahn’s algorithm or DFS on unique characters [210, 216].

### Variation 2: DFS-Based Topological Sort
- Performs DFS with a **3-State Visited Map**:
  - `0`: Unvisited
  - `1`: Visiting (currently in active recursion stack)
  - `2`: Fully Visited
- **Cycle Detection**: If DFS encounters a vertex in state `1` (Visiting), a **back-edge cycle** exists [213, 216].
- Vertices are appended to the result list in **postorder** (upon completing all outgoing edges) and reversed at the end [213, 216].

---

## 4. Edge Cases & Debugging Protocols

### A. High-Frequency Pitfalls
1. **Reversing Edge Direction**: Misinterpreting prerequisite pair $[A, B]$ direction (does $A \rightarrow B$ or $B \rightarrow A$?) reverses dependency logic [211].
2. **Missing Disconnected Components**: Initializing `in_degree` map only for vertices present in edge lists. Isolated vertices with zero edges must still be initialized in `in_degree` [209, 211].
3. **Omitting Cycle Check**: Assuming input is guaranteed to be a DAG [210, 212]. Always verify `len(result) == V` [209, 210].
