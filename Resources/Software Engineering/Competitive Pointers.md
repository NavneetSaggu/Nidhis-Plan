# Competitive Pointers

- Always use push_back to append to a string, + can give TLE
- Do not declare variables as global,pass through function. You can get TLE when doing that.
    
    Why?
    Fastest storage are the registers, followed by the caches, followed by the RAM. The thing about local variables is that the compiler optimizes them to be allocated from the registers if possible, or from the cache if not. This is why local variables are faster.