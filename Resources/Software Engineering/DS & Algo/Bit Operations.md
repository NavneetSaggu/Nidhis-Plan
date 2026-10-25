# Bit Operations

A collection of bit manipulation problems
[https://leetcode.com/problems/sum-of-two-integers/discuss/84278/A-summary%3A-how-to-use-bit-manipulation-to-solve-problems-easily-and-efficiently](https://leetcode.com/problems/sum-of-two-integers/discuss/84278/A-summary%3A-how-to-use-bit-manipulation-to-solve-problems-easily-and-efficiently) [https://leetcode.com/problems/sum-of-two-integers/discuss/132479/Simple-explanation-on-how-to-arrive-at-the-solution](https://leetcode.com/problems/sum-of-two-integers/discuss/132479/Simple-explanation-on-how-to-arrive-at-the-solution)

## How Does Bit Encoding Work?

- 1s compliment is just switching bits
- 2s compliment is 1s compliment + 1
- All -ve numbers are 2s compliment, their MSB is 1 (LSB is rightmost, MSB is leftmost)

For intuition, say we have an 8-bit encoding

```cpp
POSITIVE RANGE
0000 0000 : 0
0000 0001 : 1
0000 0010 : 2
0000 0011 : 3
0000 0100 : 4
.
.
0111 1110 : 126
0111 1111 : 127
				
NEGATIVE RANGE
1111 1111 : -1
1111 1110 : -2
1111 1101 : -3
1111 1100 : -4
.
(goes backwards)
.
1000 0001 : -127
1000 0000 : -128 (the one extra magnitude, compared to +ves)
```

## To Find Which LSB Is Set To 1

This is used extensively in Fenwick Trees

```cpp
Returns the value of the least significant bit (LSB)
  lsb(108) = lsb(0b1101100) =     0b100 = 4
  lsb(104) = lsb(0b1101000) =    0b1000 = 8
  lsb(96)  = lsb(0b1100000) =  0b100000 = 32
  lsb(64)  = lsb(0b1000000) = 0b1000000 = 64			
x = x & (-x);
```

## To Divide, Multiply By 2

```cpp
x = x>>1; // divides by 2
x = x<<1; // multiplies by 2
```