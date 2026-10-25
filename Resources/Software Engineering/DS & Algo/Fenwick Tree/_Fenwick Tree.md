# Fenwick Tree

# Uses

Fenwick trees help you resolve each of the following queries in O(logN) time

- Update value of `A[i]`
- Find sum of `A[0] + A[1] .. A[i-1] + A[i]`

Keep in mind that Fenwick trees are indexed by 1, and all range query answers are inclusive of extremeties.

# Basic Concept

Remember how in segment trees we have up to `4*N` nodes, where each node tells us the range query for a given range?

Here also, we have `N` parent nodes, each telling us the answer to the range query for a range of indices it is “responsible” for.

# Intuition

The position of the least significant bit (LSB) determines the range of responsibility that cell has to the cells below itself. 

<aside>
💡

**How To Find LSB Of Number Using Bit Operations**
`x = x & (-x)` Returns the value of the least significant bit (LSB)

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

-x is 1's compliment of x + 1

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

lsb(108) = lsb(0b1101100) =     0b100 = 4
lsb(104) = lsb(0b1101000) =    0b1000 = 8
lsb(96)  = lsb(0b1100000) =  0b100000 = 32
lsb(64)  = lsb(0b1000000) = 0b1000000 = 64			
x = x & (-x);
```

</aside>

Here, the blue bars (going downwards from the parent) represent the range of cells that each parent is responsible for.

![Screenshot 2023-06-24 at 12.37.05 AM.png](Fenwick%20Tree/Screenshot_2023-06-24_at_12.37.05_AM.png)

- For index `12` which has a binary of `1100` , LSB is 4, so it is responsible for 4 cells below itself.
- For index `12` which has a binary of `1100` , LSB is 4, so it is responsible for 4 cells below itself.

and so on..

# Range Queries

- This works on the (amongst many other principals) principal that any number can be expressed as a sum of powers of two.
- We will subtract the LSB from our index to cascade down to the next level, who’s values we will sum up.
- Complexity is `O(logN)`

![Screenshot 2023-06-24 at 12.41.32 AM.png](Fenwick%20Tree/Screenshot_2023-06-24_at_12.41.32_AM.png)

So, for example, to calculate the prefix sum of 15, we’ll add the tree values of

```cpp
tree[15] = tree[15] 
					+ tree[14] 
					+ tree[12]
					+ tree[8]
```

The code for this is quite simple

```cpp
prefixSum(i) 
		sum := 0
		while(i > 0)
				sum += tree[i]
				i := i - LSB(i)
		return sum
```

# Point Updates

- We want to update all the parents that are respnsible for the given index that is being updated.
- Point updates are the opposite of range queries, we want to add the LSB to cascade up to all the parents that are responsible for us.
- Complexity is `O(logN)`

![Screenshot 2023-06-24 at 12.50.26 AM.png](Fenwick%20Tree/Screenshot_2023-06-24_at_12.50.26_AM.png)

So, for example, to update the value at index  `9` (say to add x to it), we’ll have to update (add x to) the values of `tree[9], tree[10], tree[12], tree[16]`

The code for this is also quite simple

```cpp
add(i, x)
		while(i < N)
				tree[i] += x
				i := i + LSB(i)
```

# Constructing The Tree

- If we construct the tree naively, we can do `N` additions in `O(NlogN)` complexity.
- But if we rely on cascading effect to update values in the parents above us, we can do this in `O(N)` as well.
- Add the update value to the first immediate cell above our index that is responsible for us and proceed upwards in indices.
- For any index i, the first immediate cell above us is given by `i := i + LSB(i)`

The code for this is also quite simple

```cpp
construct(nums[])
		N = nums.size()
		tree[] = deepCopy(nums)
		for(i := 1 -> N)
				// j is the index of the first immediate parent above i
				j := i + LSB(i)
				// only update if j is in range
				if(j < N) 
							tree[j] = tree[j] + tree[i]

		return tree
```

# 2D Fenwick Tree

<aside>
💡 If you’re ever stuck with making a 2D Fenwick tree, just do the `O(NMlogNlogM)` update and be done with it. That also works just fine.

</aside>

Given a 2D matrix `matrix`, handle multiple queries of the following types:

1. **Update** the value of a cell in `matrix`.
2. Calculate the **sum** of the elements of `matrix` inside the rectangle defined by its **upper left corner** `(row1, col1)` and **lower right corner** `(row2, col2)`.

Implement the NumMatrix class:

- `NumMatrix(int[][] matrix)` Initializes the object with the integer matrix `matrix`.
- `void update(int row, int col, int val)` **Updates** the value of `matrix[row][col]` to be `val`.
- `int sumRegion(int row1, int col1, int row2, int col2)` Returns the **sum** of the elements of `matrix` inside the rectangle defined by its **upper left corner** `(row1, col1)` and **lower right corner** `(row2, col2)`.

```cpp
class NumMatrix {
    int N; 
    int M;

    // 1 indexed tree matrix
    vector<vector<int>> tree;

    // 1 indexed coordinates expected
    int prefixSum(int x, int y) {
        int sum = 0;
        for(int i=x; i>0; i -= (i & (-i))) {
            for(int j=y; j>0; j -= (j & (-j))) {
                sum += tree[i][j];
            }
        }
        return sum;
    }

    // 1 indexed coordinates expected
    void add(int x, int y, int val) {
        for(int i=x; i <= N; i += (i & (-i))) {
            for(int j=y; j <= M; j += (j & (-j))) {
                tree[i][j] += val;
            }
        }
        return;
    }

    void build(vector<vector<int>>& matrix) {
        N = matrix.size();
        M = matrix[0].size();
        tree.resize(N+1, vector<int>(M+1, 0));
        for(int i=0; i<N; i++) {
            for(int j=0; j<M; j++)
                add(i+1, j+1, matrix[i][j]);
        }
        return;
    }

public:
    NumMatrix(vector<vector<int>>& matrix) {
        build(matrix);
    }
    
    // 0 indexed expected
    int sumRegion(int row1, int col1, int row2, int col2) {
        return prefixSum(row2 + 1, col2 + 1) 
            - prefixSum(row2 + 1, col1) 
            - prefixSum(row1, col2 + 1)
            + prefixSum(row1, col1);
    }

    // 0 indexed expected
    void update(int row, int col, int val) {
        int oldVal = sumRegion(row, col, row, col);
        int updateBy = val - oldVal;
        add(row+1, col+1, updateBy);
    }
};

/**
 * Your NumMatrix object will be instantiated and called as such:
 * NumMatrix* obj = new NumMatrix(matrix);
 * obj->update(row,col,val);
 * int param_2 = obj->sumRegion(row1,col1,row2,col2);
 */
```