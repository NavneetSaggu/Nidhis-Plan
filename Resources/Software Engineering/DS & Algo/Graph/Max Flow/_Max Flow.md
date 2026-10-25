# Max Flow

- A flow graph is a directed graph where each directed edge (also called an arc) has a certain maximum capacity. Flow through each edge must be less than or equal to it’s capacity.
*Consider this analogous to a traffic system, or a network of water pipelines.*
- The capacity is a non negative value.
- Edges also have a flow value; how many units of flow are passing through that edge.
- We denote both these values adjacent to the edges, as flow / capacity.
- Initially flow is zero for all edges everywhere until we run a max flow algorithm.
- There are also two special nodes, a source (s) and a sink (t)

# The Max Flow Problem

**With an inifinite input source, how much flow can we push through the network given that each edge has a certain capacity?**

![Screenshot 2023-01-30 at 11.20.14 AM.png](Max%20Flow/Screenshot_2023-01-30_at_11.20.14_AM.png)

Effectively, max flow is the bottleneck value for the amount of flow that can pass through a network.

There can be multiple ways of achieving max flow, by giving each edge different values, but max flow will be same.

# Ford Fulkerson Method

To find max flow, the FF algorithm repeatedly finds **augmenting paths** through the **residual graph** and **augments the flow** until now more augmenting paths can be found.

The sum of the bottlenecks found in each augmenting path is the max-flow of the graph

### Augmenting Path

Is a path of edges in the residual graph with unused capacity greater than zero from the source s to the sink t.

![Screenshot 2023-01-30 at 11.46.03 AM.png](Max%20Flow/Screenshot_2023-01-30_at_11.46.03_AM.png)

The highlighted path here represents one possible augmenting path.

- Note that this path can only flow through edges that aren’t fully saturated yet.
- You know you’ve achieved maximum flow when there are no more augmenting paths that can be found.
- There can be many ways to finding augmenting path, like DFS. A more time efficient approach uses BFS to find augmenting path, this is called the Edmonds-Karp algorithm.

### Bottlneck

In any augmenting path, the bottleneck is the smallest remaining capacity (capacity - current flow) of an edge on the path. We can use this value to augment the flow along the path. 

In the path given above, the bottleneck is 6.

### Augmenting A Path

This simply means updating the flow values of all the edges along the augmenting path.

1. For forward edges, this means increasing the flow by the bottleneck value.
    
    ![Screenshot 2023-01-30 at 11.56.35 AM.png](Max%20Flow/Screenshot_2023-01-30_at_11.56.35_AM.png)
    
2. And decreasing the flow along each residual edge by the bottleneck value.
    
    ![Screenshot 2023-01-30 at 11.58.27 AM.png](Max%20Flow/Screenshot_2023-01-30_at_11.58.27_AM.png)
    
- Here’s how I understood residual edges; the pipe can now have water flowing in the opposite direction, with as much flow as is allowed such that there is no negative flow.e
- Here’s what William Fiset said, “Residual edges exist to undo bad augmenting paths that do not lead to maximum flow”.

### Residual Graph

- The residual graph is the graph which also contains residual edges. In general, we will consider residual edges to be a part of the flow graph.
- You can think of every edge in the original graph as having a residual edge with a flow / capacity of 0 / 0 initially.

# Edmonds-Karp Algorithm

- This works by finding the shortest augmenting path at each step using BFS.
- The shortest path here, means the path with the fewest edges.

The worst case complexity for

- FF with DFS is O(Ef) where f is the max flow value (can be very large)
- FF with BFS is O(VE^2)

### Intuition

DFS can lead to zig-zagging through the flow graph to find the sink; which can cause longer augmenting paths. 

Longer paths increase the chances of a smaller bottleneck increasing the overall runtime, and are thus undesirable.

### The Algorithm Implementation

The BFS runs by considering all edges having non-zero remaining capacity as being reachable.

Make the graph sparse for implementation.

```cpp
#include <iostream>
#include <limits.h>
#include <queue>
#include <string.h>
using namespace std;
 
// Number of vertices in given graph
#define V 6
 
/* Returns true if there is a path from source 's' to sink
  't' in residual graph. Also fills parent[] to store the
  path */
bool bfs(int rGraph[V][V], int s, int t, int parent[])
{
    // Create a visited array and mark all vertices as not
    // visited
    bool visited[V];
    memset(visited, 0, sizeof(visited));
 
    // Create a queue, enqueue source vertex and mark source
    // vertex as visited
    queue<int> q;
    q.push(s);
    visited[s] = true;
    parent[s] = -1;
 
    // Standard BFS Loop
    while (!q.empty()) {
        int u = q.front();
        q.pop();
 
        for (int v = 0; v < V; v++) {
            if (visited[v] == false && rGraph[u][v] > 0) {
                // If we find a connection to the sink node,
                // then there is no point in BFS anymore We
                // just have to set its parent and can return
                // true
                if (v == t) {
                    parent[v] = u;
                    return true;
                }
                q.push(v);
                parent[v] = u;
                visited[v] = true;
            }
        }
    }
 
    // We didn't reach sink in BFS starting from source, so
    // return false
    return false;
}
 
// Returns the maximum flow from s to t in the given graph
int fordFulkerson(int graph[V][V], int s, int t)
{
    int u, v;
 
    // Create a residual graph and fill the residual graph
    // with given capacities in the original graph as
    // residual capacities in residual graph
    int rGraph[V]
              [V]; // Residual graph where rGraph[i][j]
                   // indicates residual capacity of edge
                   // from i to j (if there is an edge. If
                   // rGraph[i][j] is 0, then there is not)
    for (u = 0; u < V; u++)
        for (v = 0; v < V; v++)
            rGraph[u][v] = graph[u][v];
 
    int parent[V]; // This array is filled by BFS and to
                   // store path
 
    int max_flow = 0; // There is no flow initially
 
    // Augment the flow while there is path from source to
    // sink
    while (bfs(rGraph, s, t, parent)) {
        // Find minimum residual capacity of the edges along
        // the path filled by BFS. Or we can say find the
        // maximum flow through the path found.
        int path_flow = INT_MAX;
        for (v = t; v != s; v = parent[v]) {
            u = parent[v];
            path_flow = min(path_flow, rGraph[u][v]);
        }
 
        // update residual capacities of the edges and
        // reverse edges along the path
        for (v = t; v != s; v = parent[v]) {
            u = parent[v];
            rGraph[u][v] -= path_flow;
            rGraph[v][u] += path_flow;
        }
 
        // Add path flow to overall flow
        max_flow += path_flow;
    }
 
    // Return the overall flow
    return max_flow;
}
 
// Driver program to test above functions
int main()
{
    // Let us create a graph shown in the above example
    int graph[V][V]
        = { { 0, 16, 13, 0, 0, 0 }, { 0, 0, 10, 12, 0, 0 },
            { 0, 4, 0, 0, 14, 0 },  { 0, 0, 9, 0, 0, 20 },
            { 0, 0, 0, 7, 0, 4 },   { 0, 0, 0, 0, 0, 0 } };
 
    cout << "The maximum possible flow is "
         << fordFulkerson(graph, 0, 5);
 
    return 0;
}
```

###