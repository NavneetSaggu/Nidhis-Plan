# Tarjan’s Articulation Point ❤️

- This doesn’t work in the case of double edges
- Given a DFS tree of a graph, a Back Edge is an edge that connects a vertex to a vertex that is discovered before it's parent.

![Untitled](Tarjan%E2%80%99s%20Articulation%20Point%20%E2%9D%A4%EF%B8%8F/Untitled.png)

![Untitled](Tarjan%E2%80%99s%20Articulation%20Point%20%E2%9D%A4%EF%B8%8F/Untitled%201.png)

For graph in Fig 1, and it’s DFS tree Fig 3, 4-2 and 3-1 are back edges.

- Presence of a back edge means presence of an alternative path in case the parent of the vertex is removed.
- Suppose a vertex u is having a child v such that none of the vertices in the subtree rooted at v have a back edge to any vertex discovered before u.
- This makes u an articulation point, since if u is removed, v subtree will lose connection with the rest of the graph.
- The converse is also true. i.e. if there is such a back edge, u is not an articulation point.
- NOTE: These are usually only calculated for directed graphs

Root has a special case in articulation point

- As root does not have any ancestors, the back edge check will always return true. So we check if the root has two or more children whose sub-graphs are not connected, then the root is an articulation point (think why, by plain logic)

```java
void DFS(int u,vector<int>& disc,vector<int>& low,vector<int>& parent,vector<pair<int,int>>& AP)
{
	static int time = 0;

	// Initially, all will be same
	disc[u] = low[u] = time;

	time+=1;

	int children = 0;

	for(int v: adj[u])
	{
		if(disc[v]==-1)	// If v is not visited
		{
			children++;
	
			parent[v] = u;
			DFS(v,disc,low,parent,bridge);

			// Since we just called DFS traversal, the vertices we just explored
			// might have come across a back edge. We want to update our current low
			// value since low value of v might have been updated after dicovery of
			// back edge 
			low[u] = min(low[u],low[v]);
			
			// If this is a root, and it has two or more children whose sub-graphs
			// are not connected, then this parent is an articulation point
			// We cannot subject the root to the usual AP check since this has no 
			// ancestors to have back edges to, and check will always be true
			if(parent[u] == -1 && children > 1)
				AP.pb(u);
			
			// If this is not root, we can subject it to the usual back edge check
			// Our low is lowered when we find a back edge to an ancestor with a lower 
			// discovery time. If we didn't end up lowering our low by finding a back
			// edge, that means there aren't back edges to already explored nodes
			// (ancestors) from our sub-graph. 
			// We have found an AP.
			if(parent[u] != -1 && childrenlow[v] > disc[u])
				AP.pb(u);
		}

		else if(v == parent[u])	// Ignore child to parent edge
					continue; 		    // We ignore this edge because we assume that u 
													  // is removed, to determine whether it's a
													  // articulation point
		else // If this is an ancestor, then we've come across a back edge
			low[u] = min(low[u],disc[v]); 
			// We must update low value since we have access to an ancestor now
			// We dont do a min(low[u],low[v]) because low only allows traversal
			// of one back edge; and that cannot give us a value lower than disc[v]
	}
}

void findAP_Tarjan()
{
	vector<int> disc(V,-1),low(V,-1),parent(V,-1);
	vector<pair<int,int>> AP;

	for(int i=0;i<V;++i)
		if(disc[i]==-1)
			DFS(i,disc,low,parent,AP);

	cout<<"Articulation Points are: \n";
	for(auto it: AP)
		cout<<it.first<<"-->"<<it.second<<"\n";
}

// THIS IS FOR AN UNDIRECTED GRAPH. FOR EVERY U->V THERE IS A V->U IN THE ADJ LIST
```