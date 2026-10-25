# Successor Graph

A **Successor Graph** is a **directed graph** in which each vertex has outdegree one, i.e., exactly one edge starts at each node. 

- A successor graph consists of one or more components
- Each of connected component contains one cycle
- All the other paths in that conected component lead to the cycle

Question for these properties of successor graph: [https://www.interviewbit.com/problems/good-graph/discussion/p/succesor-graph-question-ac/278784/2845/](https://www.interviewbit.com/problems/good-graph/discussion/p/succesor-graph-question-ac/278784/2845/)

## Optimal way to calculate walks

**Successor graphs** are sometimes called **functional graphs**. This is because a **successor graph** corresponds to a function that defines the edges of the graph.
Eg. succ(1) = 1, succ(2) = 1, defines a graph with edges 1→1 and 2→2 

Since each node of a successor graph has a unique successor, a function **succ(x, k)** can also be defined that gives the node when a traversal begin at **node x** and walk **k steps** forward.

A straightforward way to calculate a value of **succ(*x, k*)** is to start at node ***x***  and walk ***k*** steps forward, which takes *O(k)* time. However, using preprocessing, any value of **succ(*x, k*)** can be calculated in only *O(logk)* time.

The idea is to precalculate all values of **succ(*x, k*)** where ***k*** is a power of two and at most ***u***, where ***u*** is the maximum number of steps we will ever walk. This can be efficiently done because we can use the following recursion:

![Untitled](Successor%20Graph/Untitled.png)

This is very similar to [Binary Lifting](../Tree/Binary%20Lifting.md) 

### Time Complexity

- Precalculating the values takes *O(n*log u)* time because *O(log u)* values are calculated for each node.
- After this, any value of **succ(*x, k*)** can be calculated by presenting the number of steps ***k*** as a sum of powers of two.