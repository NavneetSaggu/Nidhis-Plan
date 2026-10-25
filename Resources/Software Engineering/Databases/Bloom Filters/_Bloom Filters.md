# Bloom Filters

[https://brilliant.org/wiki/bloom-filter/#:~:text=A bloom filter is a,is added to the set](https://brilliant.org/wiki/bloom-filter/#:~:text=A%20bloom%20filter%20is%20a,is%20added%20to%20the%20set).

- Probabilistic data structure used to test whether an element is a member of a set.
- Space-efficient and allow for very fast membership tests, but with a small probability of false positives.

![Untitled](Bloom%20Filters/Untitled.png)

In this case, the final bit array will be 1 0 1`

1. A Bloom filter consists of a bit array of size m and k hash functions, where k is the number of hash functions used to map the element to m bits in the array. 
2. To add an element to the Bloom filter, its k hash values are computed, and the corresponding bits in the array are set to 1. 
3. To test whether an element is a member of the set, its k hash values are computed.
    1. If all the corresponding bits in the array are set to 1, the element is considered to be in the set.
    2. If any of the bits are not set, the element is definitely not in the set.

The key advantage of Bloom filters is their space efficiency. The size of the bit array m is typically much smaller than the number of elements in the set, and the number of hash functions k can be adjusted to control the probability of false positives. 

The probability of false positives increases as the number of elements in the set increases, and as the number of hash functions k decreases.

Bloom filters are commonly used in applications such as caching, spell-checking, and network routers to quickly test whether a requested item is in a set without having to perform an expensive lookup operation. However, they are not suitable for applications where false positives are not acceptable, such as cryptographic applications or database indexing.