# Permutation Graph

## **Permutation Graph and Graph Decomposition**

The permutation graph theory clams that given a pair of permutation, e.g, `group1`:`a,b,a,c,a,d,c` and `group2`:`b,c,d,a,a,c,a`, if we connect the edge from group1 to group2, we can represent this pair of permutation by a graph. Based on observation, if at one position `i`, we have `group1[i]=group[i]`, then this vertex should have a self-connected edge.

![](https://miro.medium.com/max/1400/1*isI1TOU8CFEsuz9s0n89cg.png)

To transform `group2` into `group1`, we need to make swapping so that we can remove edges between two vertices(in other words, let them become self-connected edges).

Then, another observation you need is that for any cycle in our graph, the minimal steps to remove all non-self-connected edges for a cycle is the number of edges minus 1, `#edges-1`. Hence, the total number of swap:#swap=∑(Ck−1)\#swap = \sum(C_k-1)#*swap*=∑(*Ck*−1) where C1,...,CkC_1, ..., C_k*C*1,...,*Ck* are the lengths of the cycles(connected components) in our graph.

If we want to minimize the total `#swap`, we need to find a way to decompose our graph into different cycles so that each cycle is as small as possible(Because the number of edges is fixed, but if the more cycles we ended with, the more times `#swap` is minus by 1).

The more general visual example is given in this post: (854. K-Similar Strings)[[https://medium.com/@jolyon129/854-k-similar-strings-7b68772217d0](https://medium.com/@jolyon129/854-k-similar-strings-7b68772217d0)]. Actually, in this question, we don't need to decompose the graph because each vertex only has 2 degrees and there is only one way to decompose the graph. I'll talk this later.

For more detailed information about cycle decomposition, you can check this [video](https://www.youtube.com/watch?v=6CwiYoEsEPI)

Those are all the prerequisite knowledge to solve this question. Let's back to our problem.