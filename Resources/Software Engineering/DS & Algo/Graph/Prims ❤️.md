# Prims ❤️

[https://www.geeksforgeeks.org/prims-minimum-spanning-tree-mst-greedy-algo-5/](https://www.geeksforgeeks.org/prims-minimum-spanning-tree-mst-greedy-algo-5/)

- Doesn’t work with directed Graphs
- Works fine with negative weight, the concept of graph cut still applies

### Steps

1. It starts with an empty MST set. 
2. Maintain two sets of vertices. 
3. The first set contains the vertices included in the MST
4. The second set contains the vertices not yet included.
5. At every step, it considers all the edges that connect the two sets and picks the minimum weight edge from these edges. 
6. It then moves the not-yet-selected-vertex of the edge to the MST set.

### Theory

The idea behind Prim’s algorithm is simple, a spanning tree means all vertices must be connected. So the two disjoint subsets (discussed above) of vertices must be connected to make a Spanning Tree. And they must be connected with the minimum weight edge to make it a Minimum Spanning Tree.

### Algorithm

1. Create a set mstSet that keeps track of vertices already included in MST.
2. Assign a key value to all vertices in the input graph
    1. Initialize all key values as INFINITE. 
    2. Assign the key value as 0 for the first vertex so that it is picked first.
3. While mstSet doesn’t include all vertices
    1. Pick a vertex u which is not there in mstSet and has a minimum key value.
    2. Include u to mstSet.
    3. Update key value of all adjacent vertices of u. To update the key values, iterate through all adjacent vertices. For every adjacent vertex v, if the weight of edge u-v is less than the previous key value of v, update the key value as the weight of u-v

Implemented using Priority Queue

### Code

```cpp
int spanningTree(int V, vector<vector<int>> adj[])
    {
        vector<bool> inMST(V, false);
        
        // each element is (u, edge traversed to get here)
        auto comp = [](const pair<int, int> &x, const pair<int, int> &y) -> bool {return x.second > y.second;};
        priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(comp)> PQ(comp);
        PQ.push({0, 0});
        
        int sum = 0;
        
        while(!PQ.empty()) {
            int u = PQ.top().first;
            int eu = PQ.top().second;
            PQ.pop();
            if(inMST[u]) {
                continue;
            }
            else {
                inMST[u] = true;
                sum += eu;
                for(auto neighbor : adj[u]) {
                    int v = neighbor[0];
                    int ev = neighbor[1];
                    PQ.push({v, ev});
                }
            }
        }
        
        return sum;
    }
```

**Complexity** 

My implementation is O(E log(E})

An implementation with decrement operation for vertices will be O(E log(V))