# Quick Select

Three ways to do quick select

# STL nth_element

```cpp
int findKthLargest(vector<int>& nums, int k) {
	nth_element(nums.begin(),nums.begin()+nums.size()-k,nums.end());
	return nums[nums.size()-k];
}
```

# Hoare Partition (More Elegant)

```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        //partition rule: >=pivot   pivot   <=pivot
        int left=0,right=nums.size()-1,idx=0;
        while(1){
            idx = partition(nums,left,right);
            if(idx==k-1) break;
            else if(idx < k-1) left=idx+1;
            else right= idx-1;
        }
        return nums[idx];
    }
    int partition(vector<int>& nums,int left,int right){//hoare partition
        int pivot = nums[left], l=left+1, r = right;
        while(l<=r){
            if(nums[l]<pivot && nums[r]>pivot) swap(nums[l++],nums[r--]);
            if(nums[l]>=pivot) ++l;
            if(nums[r]<=pivot) --r;
        }
        swap(nums[left], nums[r]);
        return r;
    }
};
```

Remember, `l` and `r` remain in their respective positions until swappable pair isn’t found
To get a feel for how it works, consider this example

```cpp
3 2 || 3 1 2 4 5 5 6 7 7 8 2 3 1 1 1 10 11 5 6 2 4 7 8 5 || 6 
3 6 3 || 1 2 4 5 5 6 7 7 8 2 3 1 1 1 10 11 5 6 2 4 7 8 || 5 2 
3 6 3 1 || 2 4 5 5 6 7 7 8 2 3 1 1 1 10 11 5 6 2 4 7 8 || 5 2 
3 6 3 5 2 || 4 5 5 6 7 7 8 2 3 1 1 1 10 11 5 6 2 4 7 || 8 1 2 
3 6 3 5 8 4 || 5 5 6 7 7 8 2 3 1 1 1 10 11 5 6 2 4 || 7 2 1 2 
3 6 3 5 8 4 5 || 5 6 7 7 8 2 3 1 1 1 10 11 5 6 2 4 || 7 2 1 2 
3 6 3 5 8 4 5 5 || 6 7 7 8 2 3 1 1 1 10 11 5 6 2 4 || 7 2 1 2 
3 6 3 5 8 4 5 5 6 || 7 7 8 2 3 1 1 1 10 11 5 6 2 4 || 7 2 1 2 
3 6 3 5 8 4 5 5 6 7 || 7 8 2 3 1 1 1 10 11 5 6 2 4 || 7 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 || 8 2 3 1 1 1 10 11 5 6 2 4 || 7 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 || 2 3 1 1 1 10 11 5 6 2 4 || 7 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 2 || 3 1 1 1 10 11 5 6 2 4 || 7 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 7 3 || 1 1 1 10 11 5 6 2 || 4 2 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 7 3 1 || 1 1 10 11 5 6 2 || 4 2 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 7 3 4 1 || 1 10 11 5 6 || 2 1 2 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 7 3 4 1 || 1 10 11 5 || 6 2 1 2 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 7 3 4 6 1 || 10 11 || 5 1 2 1 2 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 7 3 4 6 5 10 || || 11 1 1 2 1 2 2 1 2 
3 6 3 5 8 4 5 5 6 7 7 8 7 3 4 6 5 10 || 11 || 1 1 2 1 2 2 1 2 
1 1 || 2 1 2 2 1 || 2 
1 1 2 || 1 2 2 1 || 2 
1 1 2 1 || 2 2 1 || 2 
1 1 2 1 2 || 2 1 || 2 
1 1 2 1 2 2 || 1 || 2 
1 1 2 1 2 2 1 || || 2 
1 1 2 1 2 2 1 || 2 || 
2 1 || 2 1 2 2 || 1 
2 1 || 2 1 2 || 2 1 
2 1 || 2 1 || 2 2 1 
2 1 || 2 || 1 2 2 1 
2 1 || || 2 1 2 2 1 
2 || 1 || 2 1 2 2 1
```

# **Lomuto Partition (nums[i] == p is ugly)**

```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        int N = nums.size();
        k = N-k;
        int start = 0, end = N, p;
        do {
            // [start+1, j] is < p
            // [j+1, m] is == p
            // [m+1, i] is > p
            p = nums[start];
            int i, m = start, j = start;
            for(i=start+1; i<end;) {
                if(nums[i] > p) {
                    i++;
                }
                else if(nums[i] == p) {
                    swap(nums[i], nums[m+1]);
                    i++, m++;
                }
                else {
                    if(j == m) {
                        swap(nums[j+1], nums[i]);
                        i++, j++, m++;
                    } else {
                        nums[j+1] = nums[i];
                        nums[i] = nums[m+1];
                        nums[m+1] = p;
                        i++, j++, m++;
                    }
                }
            }
            swap(nums[start], nums[j]);
            // now [j..m] is all p
            if(m < k) {
                start = m + 1;
            } else if(j > k) {
                end = j;
            } else {
                break;
            }
        } while(true);
        return p;
    }
};
```