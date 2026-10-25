# Top Sort ❤️

[https://www.geeksforgeeks.org/topological-sorting/](https://www.geeksforgeeks.org/topological-sorting/)

## DFS Algorithm

1. Pick unvisited node
2. Do DFS from this node, visiting only unvisited nodes
3. On recursive call back, add current node to top order in reverse (at front)
4. Repeat for all unvisited nodes
5. Reverse this stack to get top ordering of vertices
6. Now to check for loop, check every edge; each parent vertex must appear before child vertex in the top sort. If and only if this is violated, there is a cycle in the graph 

## BFS Algorithm (Kahn’s Algorithm)

1. We will have the indegree array to count, which nods should be visited first.

2. We will have a queue to push the nodes that don't have any dependencies.

```cpp
		vector<vector<int>> adj(numCourses);
    vector<int> indegree(numCourses, 0);
    for (auto& p : prerequisites) {
        indegree[p[1]]++;
        adj[p[0]].push_back(p[1]);
    }
    queue<int> q;
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) 
						q.push(i);
		}
    int prereq = 0;
    while (!q.empty()) {
        int el = q.front();
        q.pop();
        prereq++;
        for (auto& next : adj[el]) {
            if (--indegree[next] == 0) {
                q.push(next);
            }
        }
    }
		// will return false if there are any cycles as all nodes won't be covered
    return prereq == numCourses;
```

If the graph has any cycles, our prereq counter will never be able to cover all the nodes in the graph, and thus the function will return false.