# Bipartite Graph

A graph in which we can color all nodes, such that no two adjacent nodes have the same color.

```cpp
bool isPossible(int u, int teamToAssign, vector<vector<int>> &adj, vector<int> &team) {
    team[u] = teamToAssign;
    for(int v : adj[u]) {
        if(team[v] == teamToAssign || (team[v] == INT_MIN && !isPossible(v, !teamToAssign, adj, team)))
            return false;
    }
    return true;
}

int Solution::solve(int A, vector<vector<int> > &B) {
    vector<vector<int>> adj(A+1, vector<int>(0));
    for(auto edge : B) {
        adj[edge[0]].push_back(edge[1]);
        adj[edge[1]].push_back(edge[0]);
    }
    vector<int> team(A+1, INT_MIN);
    for(int a=1; a<=A; a++) {
        if(team[a] == INT_MIN && !isPossible(a, 0, adj, team))
            return false;
    }
    return true;
}
```

For reference, the question was: 

There are **A** people numbered 1 to A in a football academy.
The coach of the academy wants to make two teams (not necessary of equal size) but unfortunately, not all people get along, and several refuse to be put on the same team as that of their enemies.

Given a 2-D array **B** of size `M x 2` denoting the enemies i.e B[i][0] and B[i][1] both are enemies of each other.

Is it possible to divide teams in such a way?