# Tarjan’s Bridge ❤️

- If edge u->v is a bridge, there is no back edge from v or the subgraph of v, to u or any ancestor of u.
This is the concept leveraged by Tarjans algorithm to find bridges in O(V+E) time
    - disc[u] will hold the discovery number; the time at which a node was dicovered during DFS traversal.
    Also called time of insertion, tin[u] q
- low[u] (at the time of it’s evaluation) will hold the lowest discovery number reachable from u (in the DFS spanning tree) by traversing maximum one back edge.
Also callel lowest time of insertion, ltin[u]
- This is usually done for undirected graphs.
- This doesn’t work in the case of double edges
- Check out Abdul Bari’s video for more explanation of meaning of low.
- Check out Tech Dose’s video for more explanation of the bridge finding algo.
- If you feel like reading, read [https://www.hackerearth.com/practice/algorithms/graphs/articulation-points-and-bridges/tutorial/](https://www.hackerearth.com/practice/algorithms/graphs/articulation-points-and-bridges/tutorial/)

```cpp
void DFS(int u,vector<int>& disc,vector<int>& low,vector<int>& parent,vector<pair<int,int>>& bridge)
{
	static int time = 0;

	// Initially, all will be same
	disc[u] = low[u] = time;

	time+=1;

	for(int v: adj[u])
	{
		if(disc[v]==-1)	// If v is not visited
		{
			parent[v] = u;
			DFS(v,disc,low,parent,bridge);

			// Since we just called DFS traversal, the vertices we just explored
			// might have come across a back edge. We want to update our current low
			// value since low value of v might have been updated after dicovery of
			// back edge 
			low[u] = min(low[u],low[v]);
			
			// Our low is lowered when we find a back edge to an ancestor with a lower 
			// discovery time. If we didn't end up lowering our low by finding a back
			// edge, that means there aren't back edges to already explored nodes
			// (ancestors) from our sub-graph. 
			// We have found a bridge.
			if(low[v] > disc[u])
				bridge.pb({u,v});
		}

		else if(v == parent[u])	// Ignore child to parent edge
					continue; 		  // We ignore this edge because we assume that this 
													// edge is removed, to determine whether it's a
													// bridge / articulation point
		else // If this is an ancestor, then we've come across a back edge
			low[u] = min(low[u],disc[v]); 
			// We must update low value since we have access to an ancestor now
			// We also use min because this could also be a cross edge
			// We dont do a min(low[u],low[v]) because low only allows traversal
			// of one back edge; and that cannot give us a value lower than disc[v]
	}
}

void findBridges_Tarjan()
{
	vector<int> disc(V,-1),low(V,-1),parent(V,-1);
	vector<pair<int,int>> bridge;

	for(int i=0;i<V;++i)
		if(disc[i]==-1)
			DFS(i,disc,low,parent,bridge);

	cout<<"Bridges are: \n";
	for(auto it: bridge)
		cout<<it.first<<"-->"<<it.second<<"\n";
}

// THIS IS FOR AN UNDIRECTED GRAPH. FOR EVERY U->V THERE IS A V->U IN THE ADJ LIST
```