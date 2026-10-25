# Binary Lifting

Helps solve these ancestor and max edge in path queries in log(n) time.

This technique asks us to maintain an up array

For every node : u 

1. up(u, 1), i.e the ancestor 1 level above 
2. up(u, 2), i.e. the ancestor 2 levels above
3. up(u, 4)
4. …
5. up(u, 2^i)
Let up[u][x] be ancestor of u that is 2^x above u

# Algorithm

up[root][i] = -1, for all i’s, since root has no ancestors

up[u][0] = parent[u], simply pass parent values in a DFS and propagate them downwards to get this.

up[u][x] = up[up[u][x-1], x-1], simply jump By 2^(x-1) twice

Pre processing to build the up array takes n*log(n) time as well

# Kth Ancestor in Tree

If you have this the up array, you can answer up(u, k) queries in O(log(n)) time, right? Because you can always write a number as a sum of powers of 2

## **Pay Attention To**

- Look at the LCA algorithm instead, that is more general and better

```cpp
class TreeAncestor {
public:
    vector<vector<int>> up; 
    TreeAncestor(int n, vector<int>& parent) {
        up.resize(n, vector<int>(32, -1));
        // up[u][x] tells us the ancestor that is 2^x levels above u
				for(int t=0; t<31; t++) {
            for(int u=0; u<n; u++) {
                if(t == 0)
                    up[u][t] = parent[u];
                else {
                    if(up[u][t-1] != -1)
                        up[u][t] = up[ up[u][t-1] ][t-1];
                }
            }
        }
    }
    
    int getKthAncestor(int node, int k) {

				// ONE WAY
        int u = node, t = 0;
        while(k && u >= 0) {
            if(k%2 == 1)
                u = up[u][t];
            k /= 2;
            t++;
        }
        return u;

				// SHORTER WAY
				for(int x = 0; x < 20 && node != -1; x++) {
            if((1<<x) & k)
                node = up[node][x];
        }
        return node;				
};
```

# LCA in Tree

This technique asks us to maintain

For every node : u 

1. up[u][x], ancestor of u, 2^x above u, using [this](Binary%20Lifting.md) method
2. entry[u] and exit[u], entry and exit times of each node in the DFS. This helps find out if nodes are ancestors of each other in O(1) time.

Here, the up[u] default value is the index of the root of the tree, and not -1, like in the kth ancestor algorithm.

This helps the LCA algorithm run more smoothly and without if checks.

## **Pay Attention To**

- You have to check if a neighbor is a parent before making recursive dfs call
- Your LCA formula will traverse up to the last node that is not an ancestor. You have to return the node above it.
- The overall LCA function

```cpp
int n, l;
vector<vector<int>> adj;

int timer;
vector<int> tin, tout;
vector<vector<int>> up;

void dfs(int v, int p)
{
    tin[v] = ++timer;
    up[v][0] = p;
    for (int i = 1; i <= l; ++i)
        up[v][i] = up[up[v][i-1]][i-1];

    for (int u : adj[v]) {
        if (u != p)
            dfs(u, v);
    }

    tout[v] = ++timer;
}

bool is_ancestor(int u, int v)
{
    return tin[u] <= tin[v] && tout[u] >= tout[v];
}

int lca(int u, int v)
{
    if (is_ancestor(u, v))
        return u;
    if (is_ancestor(v, u))
        return v;
    for (int i = l; i >= 0; --i) {
        if (!is_ancestor(up[u][i], v))
            u = up[u][i];
    }
    return up[u][0];
}

void preprocess(int root) {
    tin.resize(n);
    tout.resize(n);
    timer = 0;
    l = ceil(log2(n));
    up.assign(n, vector<int>(l + 1));
    dfs(root, root);
}
```

# Max Edge Queries

```cpp
void dfs(int u, int p, int w, int &t, int l, vector<vector<int>> &up, vector<vector<int>> &maxEdge, 
            vector<vector<vector<int>>> &adj, vector<int> &level, vector<int> &tin, vector<int> &tout) {
    up[u][0] = p;
    maxEdge[u][0] = w;
    for(int x=1; x<20; x++) {
        maxEdge[u][x] = max(maxEdge[u][x-1], maxEdge[up[u][x-1]][x-1]);
        up[u][x] = up[up[u][x-1]][x-1];
    }
    tin[u] = t++;
    level[u] = l;
    for(auto neighbor: adj[u]) {
        if(neighbor[0] != p)
            dfs(neighbor[0], u, neighbor[1], t, l+1, up, maxEdge, adj, level, tin, tout);
    }
    tout[u] = t++;
    return;
}

// tells if u is ancestor of v i.e. there is a "downward" path form u to v
bool isAncestor(int u, int v, vector<int> &tin, vector<int> &tout) {
    return tin[u] < tin[v] && tout[u] > tout[v]; 
}

int lca(int u, int v, vector<vector<int>> &up, vector<int> &tin, vector<int> &tout) {
    if(isAncestor(u, v, tin, tout))
        return u;
    if(isAncestor(v, u, tin, tout))
        return v;
    for(int x=19; x>=0; x--)
        if(!isAncestor(up[u][x], v, tin, tout))
            u = up[u][x];
    return up[u][0];
}

int maxEdgeUp(int u, int k, vector<vector<int>> &up, vector<vector<int>> &maxEdge) {
    int ans = INT_MIN;
    for(int x=0; x<20; x++) {
        if((1<<x) & k) {
            ans = max(ans, maxEdge[u][x]);
            u = up[u][x];
        }
    }
    return ans;
}

vector<int> Solution::solve(vector<vector<int> > &A, vector<vector<int> > &B) {
    int N = A.size() + 1;
    vector<vector<vector<int>>> adj(N, vector<vector<int>>(0));
    for(auto edge : A) {
        int u = edge[0]-1;
        int v = edge[1]-1;
        int w = edge[2];
        adj[u].push_back({v, w});
        adj[v].push_back({u, w});
    }
    
    vector<vector<int>> up(N, vector<int>(20, 0));
    vector<vector<int>> maxEdge(N, vector<int>(20, INT_MIN));
    vector<int> tin(N, INT_MIN);
    vector<int> tout(N, INT_MIN);
    vector<int> level(N, INT_MIN);
    int time = 0;
    
    dfs(0, 0, INT_MIN, time, 0, up, maxEdge, adj, level, tin, tout);
    
    int Q = B.size();
    vector<int> ans(Q, INT_MIN);
    for(int q=0; q<Q; q++) {
        int u = B[q][0]-1;
        int v = B[q][1]-1;
        int b = lca(u, v, up, tin, tout);
        
        int ul = level[u];
        int vl = level[v];
        int bl = level[b];
        
        ans[q] = max(maxEdgeUp(u, ul-bl, up, maxEdge), maxEdgeUp(v, vl-bl, up, maxEdge));
    }
    
    return ans;
}
```