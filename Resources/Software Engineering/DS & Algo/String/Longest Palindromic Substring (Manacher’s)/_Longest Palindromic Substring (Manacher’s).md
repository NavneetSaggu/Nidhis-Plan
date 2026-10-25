# Longest Palindromic Substring (Manacher’s)

Best Video Resource: [https://www.youtube.com/watch?v=nbTSfrEfo6M&ab_channel=IDeserve](https://www.youtube.com/watch?v=nbTSfrEfo6M&ab_channel=IDeserve)

### The Gist

Here, we’re calculating only odd lengthed palindromes. This can be generalized to even lengthed as well.

![Screenshot 2022-10-27 at 6.56.04 PM.png](Longest%20Palindromic%20Substring%20(Manacher%E2%80%99s)/Screenshot_2022-10-27_at_6.56.04_PM.png)

Suppose you’re in a window of palindrome str[L .. R], where the substring str[L .. R] is a palindrome, you have values for L to centre of this window, and you’re calculating values for the second half of L→R. Suppose we’re calculating for B.

1. if len[mirror] goes beyond L
? = R-B’s index
2. if len[mirror] is within L
? = len[mirror]
3. Expand beyond the minimum length obtained from 1 or 2

### Code

```java
public class Manacher {
 
    public Manacher(String s) {
        
				// Create a new array with a start character, 
				// end character, and gap characters
				// "a a b a a" becomes 
				// "$ # a # a # b # a # a # @"
				
				// This lets us use the algorithm for odd lenghted
				// palindromes to also find even lengthed palindromes
				
				// All indices are divided by two in the end to 
				// be used for the original string s
        char[] T = new char[s.length()*2 + 3];
        T[0] = '$';
        T[s.length()*2 + 2] = '@';
        for (int i = 0; i < s.length(); i++) {
            T[2*i + 1] = '#';
            T[2*i + 2] = s.charAt(i);
        }
        T[s.length()*2 + 1] = '#';
         
        // P[i] will hold right spans of longest palindromes
				// centred at index 'i' for each i
				// For a palindrome of length 3, sight span will be 1
        int[]  P = new int[T.length];
        int center = 0, right = 0;
        
				// Calculate P[i] for each possible centre
				// Centre can be a character from s or a 'gap'
        for (int i = 1; i < T.length-1; i++) {
						
						// Finds the index of the mirror
						// character in the current window
            int mirr = 2*center - i;
						
						// If the current index is within the bounds of
						// the main palindrome window, follow steps 1.2.3. 
						// from The Gist to find minumum lengths
            if (i < right)
                P[i] = Math.min(right - i, P[mirr]);
	          
						// Expand the palindrome size
						// This will never overflow, thanks to the start & end characters
	          while (T[i + (1 + P[i])] == T[i - (1 + P[i])])
                P[i]++;
						
						// If the palindrome centred at 'i' expands beyond the 
						// main palindrome, update the main palindrome
            if (i + P[i] > right) {
                center = i;
                right = i + P[i];
            }
        }
         
        int length = 0;   // length of longest palindromic substring
        center = 0;   // center of longest palindromic substring
        for (int i = 1; i < P.length-1; i++) {
            if (P[i] > length) {
                length = P[i];
                center = i;
            }
        }
        System.out.println(s.substring((center - 1 - length) / 2, 
												(center - 1 + length) / 2));
 
    }
     
    public static void main(String[] args) {
        String s = "abababa";
        Manacher manacher = new Manacher(s);
    }
}
```