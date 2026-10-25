# Merge Sort Trees

The key idea is to build a segment tree with a vector at every node and the vector contains all the elements of the sub-range in a sorted order. And if we observe this segment tree structure this is somewhat similar to the tree formed during the merge sorting (that is why it is called merge sort tree) 

# Application

Given an array where each element is a vector containing integers in sorted order. The task is to answer following queries:

count(start, end, k): count the numbers smaller than or equal to k in range from array index 'start' to 'end'.

## Range Frequency Query

Design a data structure to find the **frequency** of a given value in a given subarray.

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