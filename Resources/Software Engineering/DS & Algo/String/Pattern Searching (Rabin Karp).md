# Pattern Searching (Rabin Karp)

- Rabin-Karp algorithm slides the pattern one by one over the string.
- Rabin Karp algorithm matches the hash value of the pattern with the hash value of the current substring of text, and if the hash values match then only it starts matching individual characters.
- So Rabin Karp algorithm needs to calculate hash values for the following strings.
    - Pattern itself
    - All the substrings of the text of length m
- The hash must be rollable. i.e. hash must be updatable when moving from i to i+1 in O(1) time.
- The hash function suggested by Rabin and Karp calculates an integer value.
- The integer value for a string is the numeric value of a string. i.e. treat string as a base 26  integer.
hash(”AAB”) = (1)*(26^2) + (1)*(26^1) + (2)*(26^0)
- The average and best-case running time of the Rabin-Karp algorithm is O(n+m), but its worst-case time is O(nm).
- We mod the hash value by some prime number ‘p’ to prevent overflows.
- When using mod, collisions can happen in hash values where h(X1) == h(X2) but X1 != X2.
Collision frequency is 1/p where p is the prime number used for mod.

```cpp
// d is the number of characters in the input alphabet
#define d 256
 
/* pat -> pattern
    txt -> text
    q -> A prime number
*/
void search(char pat[], char txt[], int q)
{
    int M = strlen(pat);
    int N = strlen(txt);
    int i, j;
    int p = 0; // hash value for pattern
    int t = 0; // hash value for txt
    int h = 1;
	 
    // The value of h would be "pow(d, M-1)%q"
    for (i = 0; i < M - 1; i++)
        h = (h * d) % q;
 
    // Calculate the hash value of pattern and first
    // window of text
    for (i = 0; i < M; i++) {
        p = (d * p + pat[i]) % q;
        t = (d * t + txt[i]) % q;
    }
 
    // Slide the pattern over text one by one
    for (i = 0; i <= N - M; i++) {
 
        // Check the hash values of current window of text
        // and pattern. If the hash values match then only
        // check for characters one by one
        if (p == t) {
            /* Check for characters one by one */
            for (j = 0; j < M; j++) {
                if (txt[i + j] != pat[j]) {
                    break;
                }
            }
 
            // if p == t and pat[0...M-1] = txt[i, i+1,
            // ...i+M-1]
 
            if (j == M)
                cout << "Pattern found at index " << i
                     << endl;
        }
 
        // Calculate hash value for next window of text:
        // Remove leading digit, add trailing digit
        if (i < N - M) {
            t = (d * (t - txt[i] * h) + txt[i + M]) % q;
 
            // We might get negative value of t, converting
            // it to positive
            if (t < 0)
                t = (t + q);
        }
    }
}
```