# Strongly Connected Components

Look at Kosaraju?

Can be thought of as self-contained cycles within a directed graph, where every vertex in a gien cycle can reach every other vertex in the same cycle. 

![Screenshot 2022-07-05 at 1.31.05 PM.png](Strongly%20Connected%20Components/Screenshot_2022-07-05_at_1.31.05_PM.png)

1. Each component has a self contained cycle.
2. For each component, there is no way to find a path that leaves the component and comes back

These properties assure us that all SCCs are unique

## Low Link Value

Low-link value of a node is the smallest [lowest] node id reachable from that node when doing a DFS (including itself).

When we say node id we mean discovery time.

Low link values are entirely dependent on the order in which we visit nodes during DFS

To prevent low link values of one SCC from contaminating those of another SCC, we use a visited stack

```cpp
// Psuedo code
const UNVISITED = -1
const n = number of nodes in graph
const g = adjacency list with directed edges

id = 0        // Used to give each node an id
sccCount = 0  // Used to count number of SCCs found

// Index i in these arrays represents node i
ids = [0, 0, 0, 0, ...]
low = [0, 0, 0, 0, ...]
onStack = [false, false, false, ...]
stack = an empty data structure

function findSccs():
	for(i : 0-> n-1):
		ids[i] = UNVISITED
	for(i : 0-> n-1):
		if(ids[i] == UNVISITED):
			dfs(i)
	return low

function dfs(u):
	stack.push(u)
	onStack[u] = true
	ids[u] = low[u] = id++

	// visit all neighbors and min low-link on callback
	for(v : g[u]) 
		if(ids[v] == UNVISITED:
			dfs(v)
		if(onStack[v])
			low[u] = min(low[u], low[v])

	// After having visited all the neighbors of 'u'
	// if we're at the start of a SCC, empty the seen 
	// stack until we're back to the start of the SCC
	if(ids[u] == low[u])
		while(true)
			node = stack.top()
			stack.pop()
			onStack[node] = false
			low[node] = ids[u]
			if(node == u)
				break
		sccCount++
```