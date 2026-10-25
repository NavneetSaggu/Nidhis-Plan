# Hamiltonian Path

Hamiltonian Path is a path in a directed or undirected graph that visits each vertex exactly once.

Naive algorithm to calculate this tries all premutations of N vertices and checks if it is a valid path or not ( O(N!) complexity )

## Most Optimal Algorithm

There is one algorithm given by Bellman, Held, and Karp which uses dynamic programming to check whether a Hamiltonian Path exists in a graph or not. 

Here's the idea

- For every subset S of vertices check whether there is a path that visits "EACH and ONLY" the vertices in S exactly once and ends at a vertex v.
- Do this for all v ϵ S.
- A path exists that visits each vertex in subset S and ends at vertex v ϵ S iff v has a neighbor w in S and there is a path that visits each vertex in set S-{v} exactly once and ends at w.
- If there is such a path, then adding the edge w-v to it will extend it to visit v and as it is already visiting every vertex in S-{v}, so the new path will visit every vertex in S.

The C++ code here uses bitmasking to represent subsets ( Learn about bitmasking [here](https://www.hackerearth.com/notes/bit-manipulation/))

<aside>
💡 VERY IMPORTANT
Note here, that even though we’re bottom upping the subsets, and we should ideally be doing it in the order of (all subsets of size 1, all subsets of size 2, … so on), doing it this way also works because of the nature of the algorithm itself.

</aside>

```java
bool check_using_dp(int adj[][MAXN], int n){
				// dp[j][i] checks if there is a path that visits each vertex 
				// in subset represented by mask i and ends at vertex j
        bool dp[MAXN][1<<MAXN]={0};
				
				// path including only a vertex ending at the same vertex is
				// possible for all vertices (path having only that vertex)
        for(int i=0; i<n; i++)
            dp[i][1<<i]=true;

				// iterate over all the subsets of the vertices
        for(int i=0; i<(1<<n); i++){
            for(int j=0; j<n; j++)
								
								// pick one vertex from that subset
                if(i&(1<<j)){
                    for(int k=0; k<n; k++)
												
												// what other vertex from that subset is adjacent
												// to the one we picked, and has a path ending at it
                        if(i&(1<<k) && adj[k][j] && k!=j && dp[k][i^(1<<j)]){
                            dp[j][i]=true;
                            break;
                        }
                }
        }

				// if any vertex has a path ending at it that includes all 
				// vertices, hamiltonian path is possible
        for(int i=0; i<n; i++)
            if(dp[i][(1<<n)-1])
                return true;
        return false;
    }
```

## Complexity

Time complexity of the above algorithm is O(2^n * n^2).