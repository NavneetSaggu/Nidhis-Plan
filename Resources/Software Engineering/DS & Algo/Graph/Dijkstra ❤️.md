# Dijkstra ❤️

[https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-greedy-algo-7/](https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-greedy-algo-7/)

- Gives length of shortest paths from the source to all vertices
- Similar to Prim’s algo for MST
- Works for Directed and Undirected
- Doesn’t work with negative weights, since it doesn't look back after closing. Use Bellman Ford for that.
    - More details about negative weights
        
        The idea that Dijkstra works on, is this: If we have a vertex in open such that its cost is minimal - by adding any positive number to any vertex - the minimality will never change. Without the constraint on positive numbers - the above assumption is not true.
        Since we do "know" each vertex which was "closed" is minimal - we can safely do the relaxation step - without "looking back". If we do need to "look back" - Bellman-Ford offers a recursive-like (DP) solution of doing so.
        
- To give path, maintain parent array; update when updating distance
- To find shortest distance/path to single vertex, break while when min path for it is found

### Algorithm

- [For Adjacency Matrix](https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-greedy-algo-7/)
    1. Create set sptSet (shortest path tree set), initially empty.
    Set of vertices whose minimum distance from the source is calculated and finalized. 
    2. Create array distance. (No use making this a heap)
    Assign distance as 0 for the source. Assign distance to all other vertices as infinite.
        1. Pick vertex u which is not there in sptSet and has minimum distance value.
    3. While sptSet doesn’t include all vertices
        1. Add u to sptSet.
        2. Update distance value of all adjacent vertices of u. 
        For every adjacent vertex v
            1. if(distance[u] + wt[u][v] < distance[v]) then distance[v] = distance[u] + wt[u][v]
- [For Adjacency List](https://www.geeksforgeeks.org/dijkstras-algorithm-for-adjacency-list-representation-greedy-algo-8/)
    1. Create a min heap distance of size V containing vertex number and distance value of the vertex. 
    2. Assign distance as 0 for the source. Assign distance to all other vertices as infinite.
    3. While distance heap is not empty
        1. Extract minimum distance vertex u from distance heap
        2. If distance[u] ≠ infinite, continue since this node has been processed at a lower distance before
        3. Update distance value of all adjacent vertices of u. 
        For every adjacent vertex v
            1. if(v is in distance heap && distance[u] + wt[u][v] < distance[v]) then distance[v] = distance[u] + wt[u][v]

### Complexity

Time Complexity 

- O(V^2) if Graph is Adjacency Matrix or Graph is Adjacency List and we don’t use Priority Queue
- O(ElogV) if Graph is Adjacency List and we use Priority Queue

### Implementation

For the optimized PQ implementation, there are two ways

1. Whenever you're visiting the neighbors of a vertex, insert them in the heap no matter if they are in the heap or not. Then, when you get the vertex with smallest distance from the heap, check if it has already been removed from the heap before. If it has, then remove this one too and continue. Otherwise, mark it as removed and visit all the neighbors.
**LOOK AT THE CODE FOR BETTER UNDERSTANDING**
- This makes the complexity E log(E). When considering big O, it becomes E log(V)
    
    Max of $E$ is $V^2$, which makes complexity $2Elog(V)$
    
1. Keep an explicit pointer to where in the heap each element is. Then you can perform the operation known as "decrease-key" on the element that you've already located.

```cpp
// code for first approach
vector <int> dijkstra(int V, vector<vector<int>> adj[], int S) {
        // dist min-heap holds {vertex, vertex distance from S}
        auto comp = [](const pair<int, int> &x, const pair<int, int> &y) -> bool {return x.second > y.second;};
        priority_queue <pair<int, int>, vector<pair<int, int>>, decltype(comp)> PQ(comp);
        vector<int> distance(V, INT_MAX);
        
        // pushing values for source
        PQ.push({S, 0});
        
        while(!PQ.empty()) {
						auto[u, d_u] = PQ.top(); PQ.pop();
            if(distance[u] != INT_MAX)
                continue;
            else {
                distance[u] = d_u;
                for(vector<int> v : adj[u]) {
                    if(distance[v[0]] == INT_MAX) {
                        PQ.push({v[0], d_u + v[1]});
                    }
                }
            }
        }
        return distance;
    }
```