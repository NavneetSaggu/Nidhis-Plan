# Floyd-Warshall ❤️

[geeksforgeeks.org/floyd-warshall-algorithm-dp-16/](http://geeksforgeeks.org/floyd-warshall-algorithm-dp-16/)

- Used to find shortest distances between every pair of vertices in a given edge weighted directed Graph.
- Works for negative edges

<aside>
💡 Make sure the outermost loop is for the intermediate vertex. The code must look like
for( every intermediate vertex )
    for( every source vertex )
        for( every destination vertex )

Make sure distance of every node from itself is set to 0

</aside>

## Steps

We initialize the solution matrix same as the input graph matrix as a first step. 

Then we update the solution matrix by considering all vertices as an intermediate vertex. The idea is to one by one pick all vertices and updates all shortest paths which include the picked vertex as an intermediate vertex in the shortest path. 

When we pick vertex number k as an intermediate vertex, we already have considered vertices {0, 1, 2, .. k-1} as intermediate vertices. 

For every pair (i, j) of the source and destination vertices respectively, there are two possible cases.

1. k is not an intermediate vertex in shortest path from i to j. We keep the value of dist[i][j] as it is.
2. k is an intermediate vertex in shortest path from i to j. We update the value of dist[i][j] as dist[i][k] + dist[k][j] if dist[i][j] > dist[i][k] + dist[k][j]

```cpp
// This is for an undirected graph
// For a directed graph, we only insert edge in one direction
int floydWarshall(int n, vector<vector<int>>& edges) {
        vector<vector<int>> distance(n, vector<int>(n, INT_MAX));

        for(auto edge : edges) {
            distance[edge[0]][edge[1]] = edge[2];
            distance[edge[1]][edge[0]] = edge[2];
        }

        for(int u=0; u<n; u++)
            distance[u][u] = 0;

        for(int x=0; x<n; x++) {
            for(int u=0; u<n; u++) {
                for(int v=0; v<n; v++) {
                    if(distance[u][x] != INT_MAX && distance[x][v] != INT_MAX &&
                        distance[u][x] + distance[x][v] < distance[u][v]) {
                        distance[u][v] = distance[u][x] + distance[x][v];
                    }
                }
            }
        }
}
```

## Intuition / Understanding How It Works

This is actually a space optimal version of the algorithm. 

The original algorithm has the a 3 dimensional dp array.

```cpp
dp[k][i][j] = min(dp[k-1][i][j], dp[k-1][i][k] + dp[k-1][k][j])
```

Here `dp[k][i][j]` represents the minimum cost path to go from `i` to `j` using nodes `0, 1, 2, 3 .. k-1, k` as intermediate nodes

To understand how this works

1. Consider how `dp[0][i][j]`  is calculated and how it is valid.
2. Think of how  `dp[0][i][j]` can be used to find `dp[1][i][j]`.
3. From thereon you can apply induction to guarantee that this gives correct result for `dp[n-1][i][j]`