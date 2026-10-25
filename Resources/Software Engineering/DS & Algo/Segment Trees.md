# Segment Trees

# Range Sum Query Segment Tree

## Complexity

- **Building Tree**: O(N)
- **Update / Querying**: O(logN)

## **Pay Attention To**

- Order of checks in each function
- The recursive call in rangeSum function

Having a segment array and going left and right by doing math operations is the same as having a pointer tree. We’re just able to do that with an integer array because we only have one attribute (an integer) to store in the node.

```cpp
// SOLUTION USING SEGMENT TREES
class NumArray {
private:
    vector<int> seg;
    int len;

    // ind tells us what node we're at within our segment tree
    void build(int ind, int low, int high, vector<int> &nums) {
        if(low == high) {
            seg[ind] = nums[low];
            return;
        }
        int mid = (low+high)/2;
        build(2*ind + 1, low, mid, nums);
        build(2*ind + 2, mid + 1, high, nums);
        seg[ind] = seg[2*ind + 1] + seg[2*ind + 2];
        return;
    }

    int rangeSum(int ind, int low, int high, int qLow, int qHigh) {
        if(low >= qLow && high <= qHigh)
            return seg[ind];
        if(high < qLow || qHigh < low)
            return 0;
        int mid = (low + high)/2;
        return rangeSum(2*ind+1, low, mid, qLow, qHigh) + 
				rangeSum(2*ind+2, mid+1, high, qLow, qHigh);
    }

    void updateSum(int ind, int low, int high, int qInd, int val) {
        if(low == high && low == qInd) {
            seg[ind] = val;
            return;
        }
        if(qInd < low || qInd > high)
            return;
        int mid = (low+high)/2;
        updateSum(2*ind+1, low, mid, qInd, val);
        updateSum(2*ind+2, mid+1, high, qInd, val);
        seg[ind] = seg[2*ind + 1] + seg[2*ind + 2];
        return;
    }

public:
    NumArray(vector<int>& nums) {
        len = nums.size();
        seg.resize(4*len + 1, 0);
        build(0, 0, len-1, nums);
    }
    
    void update(int index, int val) {
        updateSum(0, 0, len-1, index, val);
    }
    
    int sumRange(int left, int right) {
        return rangeSum(0, 0, len-1, left, right);
    }
};

/**
 * Your NumArray object will be instantiated and called as such:
 * NumArray* obj = new NumArray(nums);
 * obj->update(index,val);
 * int param_2 = obj->sumRange(left,right);
 */
```

# Persistent Segment Tree for **Range Frequency Queries**

Similar tree can be implemented for other use cases where the entire array needs to be stored.

```cpp
class SegmentTree {
public:
    vector<vector<pair<int,int>>> S;
    int bSearch(vector<pair<int,int>> &arr, int x) {
        int l = 0, h = arr.size()-1, m;
        while(l <= h) {
            m = (l + h)/2;
            if(arr[m].first == x)
                return arr[m].second;
            else if(arr[m].first < x)
                l = m+1;
            else
                h = m-1;
        }
        return 0;
    }

    void buildTree(int ind, int left, int right, vector<int> &arr) {
        if(left == right) {
            S[ind] = {{arr[left], 1}};
            return;
        }
        int mid = (left + right)/2;
        buildTree(2*ind + 1, left, mid, arr);
        buildTree(2*ind + 2, mid + 1, right, arr);
        int Nl = S[2*ind + 1].size(), Nr = S[2*ind + 2].size(), l = 0, r = 0, m = 0;
        while(l < Nl && r < Nr) {
            if(S[2*ind + 1][l].first < S[2*ind + 2][r].first)
                S[ind].push_back(S[2*ind + 1][l++]);
            else if(S[2*ind + 1][l].first > S[2*ind + 2][r].first)
                S[ind].push_back(S[2*ind + 2][r++]);
            else {
                S[ind].push_back({S[2*ind + 1][l].first, 
                    S[2*ind + 1][l++].second + S[2*ind + 2][r++].second});
            }
        }
        while(l < Nl) 
            S[ind].push_back(S[2*ind + 1][l++]);
        while(r < Nr)
            S[ind].push_back(S[2*ind + 2][r++]);
    }

    SegmentTree(vector<int> &arr) {
        int N = arr.size();
        S.resize(4*N + 1, vector<pair<int,int>>(0));
        buildTree(0, 0, N-1, arr);
        return;
    }

    SegmentTree() {
    }

    int query(int ind, int qLeft, int qRight, int left, int right, int x) {
        if(qRight < left || qLeft > right)
            return 0;
        else if(qLeft <= left && right <= qRight)
            return bSearch(S[ind], x);
        int mid = (left+right)/2;
        return query(2*ind + 1, qLeft, qRight, left, mid, x) +
            query(2*ind + 2, qLeft, qRight, mid + 1, right, x);
    }
};

class RangeFreqQuery {
public:
    SegmentTree sTree;
    int N;
    RangeFreqQuery(vector<int>& arr) {
        sTree = SegmentTree(arr);
        N = arr.size();
    }
    
    int query(int left, int right, int value) {
        return sTree.query(0, left, right, 0, N-1, value);
    }
};

/**
 * Your RangeFreqQuery object will be instantiated and called as such:
 * RangeFreqQuery* obj = new RangeFreqQuery(arr);
 * int param_1 = obj->query(left,right,value);
 */
```