# Time Complexities

![Untitled](Time%20Complexities/Untitled.png)

# Ammortized Time

Amortized time is the way to express the time complexity when an algorithm has the very bad time complexity only once in a while besides the time complexity that happens most of time.

Think of it average time taken per operation, if you do many operations.

> Once it happens, it won’t happen again for so long that the cost is “amortized.” -*Cracking the Coding Interview 6th edition 43*
> 

## Example

In a vector / ArrayList, insertion takes O(n) when the capacity has been reached, but O(1) time normally. Thus, the amortized time for each insertion is O(1).