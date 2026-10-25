# Union Find ❤️

- Union Find is a data structure that keeps track of elements that are split into one or more disjoint sets.
- It has two primary operations, union and find.

## Functions

- find(x) will tell you what group x belongs to
- union(x,y) will merge groups of x and y together

## Uses

- Where is this used?
    - Kruskal’s MST algorithm
        
        Ofcourse, because all nodes are slowly joining together to become tree.
        
    - Grid percolation
        
        There’s a bunch of dots on the grid and we’re joining dots to see if there’s a path from x to y
        
    - Network connectivity
        
        Are two vertices in a graph connected to each other through a series of edges?
        
    - Can detect cycle in an undirected graph, not in a directed graph though.

## Complexity

| Construction | O(n) |
| --- | --- |
| Union | a(n) |
| Find | a(n) |
| Get Component Size | a(n) |
| Check If Connectec | a(n) |
| Count Components | O(1) |

a(n) is ammortized ([_Time Complexities](_Time%20Complexities.md)) constant time i.e. O(1)

## Create Union Find (Not Path Compressed)

BEST REFERENCE GUIDE FOR UNION FIND: [https://www.hackerearth.com/practice/notes/disjoint-set-union-union-find/](https://www.hackerearth.com/practice/notes/disjoint-set-union-union-find/)

1. Construct a bijection (a mapping) between your objects and the integers in the range [0,n). Make this a hash map.
    
    ```cpp
    E -> 0
    F -> 1
    I -> 2
    D -> 3
    C -> 4
    A -> 5
    ...
    ```
    
    (We’re making an array based UF which is very efficient and easy to work with)
    
2. We can assign arbitrary mapping to each object, so long as each object maps to a unique number (a random bijection)
3. Create an array, where each index is associated to an object (done through our mapping)
    
    ```cpp
      E   F   I   D   C   A   ...
    | 0 | 1 | 2 | 3 | 4 | 5 | ... 
    	0   1   2   3   4   5   ...
    
    parent of index i is going to be whatever index is at position i
    parent[i] = parent[parent[i]]
    ```
    
4. If I do a union(F,D), this will become
    
    ```cpp
      E   F   I   D   C   A   ...
    | 0 | 1 | 2 | 1 | 4 | 5 | ... 
    	0   1   2   3   4   5   ...
    ```
    
5. Each time we merge two groups, let’s merge smaller to larger
So union(A,D) will become
    
    ```cpp
      E   F   I   D   C   A   ...
    | 0 | 1 | 2 | 1 | 4 | 3 | ... 
    	0   1   2   3   4   5   ...
    ```
    

## Operations

### Find Operation

To find which component a particular element belongs to, find the root of that component by following the parent nodes until a self loop is reached (a node whos’ parent is itself)

### Union Operation

To unify two elements, find which are the root nodes of each component and if the root nodes are different make one of the root nodes be the parent of the other

<aside>
🌻 THIS IS A NON PATH-COMPRESSED VERSION OF UNION FIND, find(A) will take two hops, and worst case can be O(V)

</aside>

## Union Find With Path Compression

Every time you iterate through children → parent path to get to the parent, make all the children point directly to the parent.

```cpp
class UnionFind {
    public:
    UnionFind(int n) : parent(n) {
        iota(parent.begin(), parent.end(), 0);
    }

    int Find(int x) {
        int temp = x;
        while (temp != parent[temp]) {
            temp = parent[temp];
        }
		// Path compression below
        while (x != temp) {
            int next = parent[x];
            parent[x] = temp;
            x = next;
        }
        return x;
    }

    void Union(int x, int y) {
        int xx = Find(x);
        int yy = Find(y);
				// we dont really need this condition
        if (xx != yy) {
            parent[xx] = yy;
        }
    }

    private:
           vector<int> parent;
};
```