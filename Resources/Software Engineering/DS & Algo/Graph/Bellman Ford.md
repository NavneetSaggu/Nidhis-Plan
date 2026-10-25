# Bellman Ford

[https://www.geeksforgeeks.org/bellman-ford-algorithm-dp-23/](https://www.geeksforgeeks.org/bellman-ford-algorithm-dp-23/)

- Gives length of shortest paths from the source to all vertices
- Works with negative weights
- Will report if there is a negative weight cycle

### Theory

Like other Dynamic Programming Problems, the algorithm calculates shortest paths in a bottom-up manner. It first calculates the shortest distances which have at-most one edge in the path. Then, it calculates the shortest paths with at-most 2 edges, and so on. After the i-th iteration of the outer loop, the shortest paths with at most i edges are calculated. There can be maximum |V| – 1 edges in any simple path, that is why the outer loop runs |v| – 1 times. 

The idea is, assuming that there is no negative weight cycle, if we have calculated shortest paths with at most i edges, then an iteration over all edges guarantees to give shortest path with at-most (i+1) edges.

Look at Bellman Ford solutions to this problem for a more intuitive understanding [https://leetcode.com/problems/cheapest-flights-within-k-stops/description/](https://leetcode.com/problems/cheapest-flights-within-k-stops/description/)

### Algorithm

***1)** This step initializes distances from the source to all vertices as infinite and distance to the source itself as 0. Create an array dist[] of size |V| with all values as infinite except dist[src] where src is source vertex.*

***2)** This step calculates shortest distances. Do following |V|-1 times where |V| is the number of vertices in given graph.*

*…..**a)** Do following for each edge u-v*

*………………If dist[v] > dist[u] + weight of edge uv, then update dist[v]*

*………………….dist[v] = dist[u] + weight of edge uv*

***3)**This step reports if there is a negative weight cycle in graph. Do following for each edge u-v*

*……If dist[v] > dist[u] + weight of edge uv, then “Graph contains negative weight cycle”*

*The idea of step 3 is, step 2 guarantees the shortest distances if the graph doesn’t contain a negative weight cycle. If we iterate through all edges one more time and get a shorter path for any vertex, then there is a negative weight cycle*

### Complexity

Time Complexity 

- O(EV), more than Dijkstra, but this is simpler than Dijkstra and is preferred in Distributed Systems.

### Code

```cpp
// edges looks like [{u,v,w}, {u,v,w} ..]
// this is a directed graph
int isNegativeWeightCycle(int V, vector<vector<int>>edges){
	    // let's find shortest path to all vertices from 0
	    // let d hold all the distances, d[i] is min distance from S to ii
	    vector<int> dist(V, INT_MAX);
	    
	    // setting distance for vertex 0
	    dist[0] = 0;
	    
	    for(int i=0; i<V-1; i++) {
	        for(int e=0; e<edges.size(); e++) {
	            int u = edges[e][0];
	            int v = edges[e][1];
	            int w = edges[e][2];
	            if(dist[u] != INT_MAX && (dist[v] > dist[u] + w))
	                dist[v] = dist[u] + w;
	        }
	    }
	    
	    // loop to check negative weight cycle
	    for(int e=0; e<edges.size(); e++) {
	        int u = edges[e][0];
	        int v = edges[e][1];
	        int w = edges[e][2];
	        if(dist[v] > dist[u] + w)
	            return true;
	    }
	    
	    return false;
	}
```