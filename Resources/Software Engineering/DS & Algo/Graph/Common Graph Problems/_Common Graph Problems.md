# Common Graph Problems

# Common Graph Problems

Questions to ask first

- Is the graph directed or undirected
- Are the edges weighted
- Are edges sparse or dense
- What representation fits best; adjacency matrix ar adjecency list?

### Shortest Path Problem

Given a weighted graph, find the shortes path of edges from node A to B

![Screenshot 2022-11-02 at 8.12.19 AM.png](Common%20Graph%20Problems/Screenshot_2022-11-02_at_8.12.19_AM.png)

- BFS (For unweighted)
- [Dijkstra ❤️](Dijkstra%20%E2%9D%A4%EF%B8%8F.md)
- [Bellman Ford](Bellman%20Ford.md)
- [Floyd-Warshall ❤️](Floyd-Warshall%20%E2%9D%A4%EF%B8%8F.md)

### Connectivity

Is there a path between node A and node B?

- [Union Find ❤️](Union%20Find%20%E2%9D%A4%EF%B8%8F.md)
- Any search, DFS or BFS

### Negative Cycles

Does my DAG have -ve cycles?

- [Bellman Ford](Bellman%20Ford.md)
- [Floyd-Warshall ❤️](Floyd-Warshall%20%E2%9D%A4%EF%B8%8F.md)

### Strongly Connected Components

Can be thought of as self-contained cycles within a directed graph, where every vertex in a gien cycle can reach every other vertex in the same cycle.

![Screenshot 2022-10-29 at 6.44.03 PM.png](Common%20Graph%20Problems/Screenshot_2022-10-29_at_6.44.03_PM.png)

- [_Strongly Connected Components](_Strongly%20Connected%20Components.md)

### Traveling Salesman Problem

Traverse all nodes exactly once, and return to the origin with the shortest possible route

- Held Karp (DP)
- Branch and Bound

### Bridge Finding

A bridge / cut edge is an edge whose removal increases the number of connected components.

These highlight weak points, bottlenecks or vulnerlabilities in graphs.

- [Tarjan’s Bridge ❤️](Tarjan%E2%80%99s%20Bridge%20%E2%9D%A4%EF%B8%8F.md)

### Articulation  Point  Finding

An articulation point or cut vertex is a vertex whose removal increases the number of connected components.

- [_Tarjan’s Articulation Point ❤️](_Tarjan’s%20Articulation%20Point%20❤️.md)

### Minimum Spanning  Tree

Subset of the edges having minimum sum that encompasses all the vertices.

- [Kruskal](Kruskal.md)
- [Prims ❤️](Prims%20%E2%9D%A4%EF%B8%8F.md)

### Network Flow

Find the maximum flow through a special type of graph, called a flow network.

In flow networks, edge weights represent capacities. Things like max cars that fit on a road, or maximum volume of water that can flow through a pipe.

With an infinite input source, how much “flow” can we push though the network? (given a source and a sink node)

These help us identify bottlenecks and fix edges that have lower capacities.