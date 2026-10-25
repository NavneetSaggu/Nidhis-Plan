# Math Concepts

# Bijection

In mathematics, a bijection (also known as a bijective function, one-to-one correspondence, or invertible function) is a function between the elements of two sets , where each element of one set is paired with exactly one element of the other set, and each element of the other set is paired with exactly one element of the first set. There are no unpaired elements.

# 🙃 Modular Inverse

## Method 1 (My Preferred, Use This, This Is Good Enough)

- This algorithm is based on Fermat’s little theorem.
Fermat’s little theorem states that if p is a prime number, then for any integer a, the number a^p – a is an integer multiple of p.
- Here p is a prime number
a^p ≡ a (mod p)      <== MAIN RESULT FROM FERMATS THAT WE USE
a^(p-1) ≡ 1 (mod p)
- This gives us
a^(m-2) ≡ a^(-1) (mod m)

```cpp
		// Function to find modular inverse of a under modulo M
		// Assumption: M is prime
		void modInverse(int A, int M)
		{
		    int g = gcd(A, M);
		    if (g != 1)
		        cout << "Inverse doesn't exist";
		    else {
		        // If a and m are relatively prime, then modulo
		        // inverse is a^(m-2) mode m
		        cout << "Modular multiplicative inverse is "
		             << power(A, M - 2, M);
		    }
		}

		// To compute x^y under modulo m
		int power(int x, unsigned int y, unsigned int M)
		{
		    if (y == 0)
		        return 1;

		    int p = power(x, y / 2, M) % M;
		    p = (p * p) % M;

		    return (y % 2 == 0) ? p : (x * p) % M;
		}

		// Function to return gcd of a and b
		int gcd(int a, int b)
		{
		    if (a == 0)
		        return b;
		    return gcd(b % a, a);
		}
```

## Method 2 (For Thoroughness)

- For any two integers ‘a’ and ‘b’, their gcd, can be expressed as a linear combination of 'a' and 'b' (here x and y are integers)
ax + by = gcd(a, b)
- We use this idea to find the multiplicative inverse of ‘A’ under ‘M’; we put b = M in the above formula. Since we know that A and M are relatively prime, we can put the value of gcd as 1.
		Ax + My = 1
- If we take modulo M on both sides, we get
		Ax  ≅ 1 (mod M)
- Thus the x coefficient in this equation will be the modular inverse of A x and y can be calculated using the Extended Euclid Algorithm. 
This just works backwards from the Euclidean Algorithm for GCD, as given in  this video <https://www.youtube.com/watch?v=hB34-GSDT3k&ab_channel=GVSUmath>

```cpp
		// Returns modulo inverse of a with respect
		// to m using extended Euclid Algorithm
		// Assumption: a and m are coprimes, i.e.,
		// gcd(A, M) = 1
		int modInverse(int A, int M)
		{
		    int m0 = M;
		    int y = 0, x = 1;

		    if (M == 1)
		        return 0;

		    while (A > 1) {
		        // q is quotient
		        int q = A / M;
		        int t = M;

		        // m is remainder now, process same as
		        // Euclid's algo
		        M = A % M, A = t;
		        t = y;

		        // Update y and x
		        y = x - q * y;
		        x = t;
		    }

		    // Make x positive
		    if (x < 0)
		        x += m0;

		    return x;
		}
```

# ❗ nCr

This is the best way for calculating nCr. No, it does not result in any ceils or floors.

```jsx
int C(long long int n, long long int r) {
    long long int ans = 1;
    for(int i=0; i<r; i++)
        ans = (ans*(n-i)/(i+1))%1000000007;
    return ans;
}
```