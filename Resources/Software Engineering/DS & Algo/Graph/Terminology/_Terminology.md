# Terminology

# Special Types Of Graphs

### Trees

Are undirected graphs with no cycles. For N nodes, these graphs have N-1 edges.

### Rooted Trees

Are directed trees with a designated root node, where all nodes either 

- point away from (out-tree / arborescence)
- point towards (in-tree / anti-arborescence)

the root node.

### Bipartite Graph

Graph whose vertices can be split into two groups, U and V. such that every edge connects between U and V.

A two colorable graph is bipartite.

A graph with no odd length cycle is bipartite.

A tree is always bipartite.

### Complete Graph

There is a unique edge between every pair of nodes. 

### Simple Path

A path in which we ignore any cycles, and there is no repition of vertices.

# DFS Tree Edges In Directed Graphs

For different algorithms, depending on how the algo iterates throught neighbors, different edges can be different kinds. 
But for a given input and input algo, which edge will be which is fixed.

![Untitled](Terminology/Untitled.png)

### Tree Edge

It is an edge which is present in the tree obtained after applying DFS on the graph. All the Green edges are tree edges.

### Forward Edge

It is an edge (u, v) such that v is descendant but the edge is not a part of the DFS tree. Edge from **1 to 8** is a forward edge.

### Back Edge

It is an edge (u, v) such that v is ancestor of node u but the edge is not part of DFS tree. Edge from **6 to 2** is a back edge.

**INTUITIVE**: Given a DFS tree of a graph, a Back Edge is an edge that connects a vertex to a vertex that is discovered before it's parent.

### Cross Edge

It is a edge which connects two node such that they do not have any ancestor and a descendant relationship between them. Edge from node **5 to 4** is cross edge.

### Cut Edge / Bridge

A bridge is an edge whose removal increases the number of connected components.