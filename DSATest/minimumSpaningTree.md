A **Minimum Spanning Tree (MST)** is a subset of the edges of a connected, `undirected graph` that connects `all the vertices` together, `without any cycles` and with the `minimum possible total edge weight`. In simpler terms, it is a tree that spans all the vertices in the graph and has the least possible sum of edge weights.

### Key Characteristics of an MST:
1. **Spanning Tree**: It includes all the vertices of the graph.
2. **Minimum Weight**: The sum of the weights of the edges in the spanning tree is minimized.
3. **No Cycles**: It does not contain any cycles, ensuring it remains a tree structure.

### Applications of MST:
- Designing network layouts (e.g., computer networks, electrical grids).
- Approximation algorithms for NP-hard problems.
- Cluster analysis in data mining.

### Algorithms to Find MST:
There are several algorithms to find the Minimum Spanning Tree of a graph, with the most common ones being:

#### 1. **Kruskal's Algorithm**:
Kruskal's algorithm sorts all the edges of the graph in increasing order of their weight and adds edges to the MST in this order, as long as they do not form a cycle.

**Steps**:
1. Sort all the edges in non-decreasing order of their weights.
2. Pick the smallest edge. Check if it forms a cycle with the spanning tree formed so far. If not, include this edge. Else, discard it.
3. Repeat step 2 until there are (V-1) edges in the spanning tree.

**Time Complexity**: O(E log E) or O(E log V), where E is the number of edges and V is the number of vertices.

#### 2. **Prim's Algorithm**:
Prim's algorithm starts with a single vertex and grows the MST one edge at a time by adding the smallest edge that connects a vertex in the MST to a vertex outside of the MST.

**Steps**:
1. Start with an arbitrary vertex and add it to the MST.
2. Find the smallest edge connecting the MST to a vertex not in the MST.
3. Add the edge and the vertex to the MST.
4. Repeat steps 2 and 3 until all vertices are included in the MST.

**Time Complexity**: O(V²) for the simple implementation using an adjacency matrix and O(E log V) with a priority queue (binary heap).

### Example:

Consider a graph with vertices A, B, C, D, and E and the following weighted edges:
- A-B: 1
- A-C: 3
- B-C: 3
- B-D: 6
- C-D: 4
- C-E: 2
- D-E: 5

Using Kruskal's algorithm:
1. Sort edges by weight: (A-B, 1), (C-E, 2), (A-C, 3), (B-C, 3), (C-D, 4), (D-E, 5), (B-D, 6).
2. Add edge A-B to MST.
3. Add edge C-E to MST.
4. Add edge A-C to MST.
5. Add edge C-D to MST.

Final MST edges: (A-B, 1), (C-E, 2), (A-C, 3), (C-D, 4). The total weight is 1+2+3+4 = 10.

Using Prim's algorithm starting from vertex A:
1. Start with vertex A.
2. Add edge A-B (weight 1).
3. Add edge A-C (weight 3).
4. Add edge C-E (weight 2).
5. Add edge C-D (weight 4).

Final MST edges: (A-B, 1), (A-C, 3), (C-E, 2), (C-D, 4). The total weight is 1+3+2+4 = 10.

Both algorithms yield the same MST with a total weight of 10.
