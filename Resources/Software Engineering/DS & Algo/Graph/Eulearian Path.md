# Eulearian Path

An algorithm that finds a path that uses every ***edge*** in a graph only once.

Remember that a directed graph has a Eulerian cycle if the following conditions are true 

1. All vertices with nonzero degrees belong to a single strongly connected component. 
2. In degree and out-degree of every vertex is the same.

The algorithm below is a solution for the "**Reconstruct Itinerary**": [https://leetcode.com/problems/reconstruct-itinerary/](https://leetcode.com/problems/reconstruct-itinerary/)

The idea here is to mark visited edges during traversal and not visited nodes.