# Pattern Searching (Z Algorithm)

Best Video Resource: [https://www.youtube.com/watch?v=CpZh4eF8QBw&t=1179s&ab_channel=TusharRoy-CodingMadeSimple](https://www.youtube.com/watch?v=CpZh4eF8QBw&t=1179s&ab_channel=TusharRoy-CodingMadeSimple)

- Linear time pattern searching algorithm
- Total time taken is O(m + n) with linear space complexity.
- In this algorithm, we construct a Z array.
    - For a string str[0 .. N-1], Z array is of same length as string.
    - An element Z[i] of Z array stores length of the longest substring starting from str[i] which is also a prefix of str[0..n-1].
    - The first entry of Z array is meaning less as complete string is always prefix of itself.

```cpp
Index            0   1   2   3   4   5   6   7   8   9  10  11 
Text             a   a   b   c   a   a   b   x   a   a   a   z
Z values         X   1   0   0   3   1   0   0   2   2   1   0
```

### **How is Z Array Helpful in Searching Pattern in Linear Time?**

- The idea is to concatenate pattern and text, and create a string “P$T” where P is pattern, $ is a special character should not be present in pattern and text, and T is text.
- Build the Z array for concatenated string.
- In Z array, if Z value at any point is equal to pattern length, then pattern is present at that point. (by the very definition of Z array).

```cpp
Pattern P = "aab",  Text T = "baabaa"

The concatenated string is = "aab$baabaa"

Z array for above concatenated string is
String           a  a  b  $  b  a  a  b  a  a
Z                x  1  0  0  0  3  1  0  2  1  

Since length of pattern is 3, the value 3 in Z array 
indicates presence of pattern.
```

### **How to Construct Z Array?**

The idea is to maintain an interval [L, R] which is the interval with max R
such that [L,R] is prefix substring (substring which is also prefix).

```cpp
vector<int> getZArray(string s) {
        int N = s.size();
        vector<int> Z(N, 0);
        int L = 0, R = 0;
        for(int i=1; i<N; i++) {
            if(i > R) {
                L = R = i;
                while(s[Z[i]] == s[L + Z[i]])
                    Z[i]++;
                R = L + Z[i] - 1;
            } else {
								// we are operating inside box
                int k = i - L;

                // if value does not stretches till  
								// right bound then just copy it.
								// (if it stretches till right bound,
								// then there might be more matched, 
								// so we must check those with the else)
                if(i + Z[k] - 1 < R)
                    Z[i] = Z[k];

								// otherwise try to see if there are 
								// more matches.	
                else {
                    L = i;
                    while(s[R] == s[R-L])
                        R++;
                    Z[i] = R - L;
                    R--;
                }
            }
        }
        return Z;
    }
```

Steps for maintaining this interval are as follows –

1. If i > R then there is no prefix substring that starts before i and
ends after i, so we reset L and R and compute new [L,R] by comparing
str[0..] to str[i..] and get Z[i] (= R-L+1).
2. If i <= R then let K = i-L, now Z[i] >= min(Z[K], R-i+1) because
str[i..] matches with str[K..] for atleast R-i+1 characters (they are in
[L,R] interval which we know is a prefix substring).
Now two sub cases arise –
a) If Z[K] < R-i+1 then there is no prefix substring starting at
str[i] (otherwise Z[K] would be larger) so Z[i] = Z[K] and
interval [L,R] remains same.
b) If Z[K] >= R-i+1 then it is possible to extend the [L,R] interval
thus we will set L as i and start matching from str[R] onwards and
get new R then we will update interval [L,R] and calculate Z[i] (=R-L+1)