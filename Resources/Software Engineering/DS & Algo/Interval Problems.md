# Interval Problems

Here I provide a concise template that I summarize for the so-called "Overlapping Interval Problem", e.g. Minimum Number of Arrows to Burst Balloons, and Non-overlapping Intervals etc. I found these problems share some similarities on their solutions.

- Sort intervals/pairs in increasing order of the start position.
- Scan the sorted intervals, and maintain an "active set" for overlapping intervals. At most times, we do not need to use an explicit set to store them. Instead, we just need to maintain several key parameters, e.g. the number of overlapping intervals (count), the minimum ending point among all overlapping intervals (minEnd).
- If the interval that we are currently checking overlaps with the active set, which can be characterized by cur.start > minEnd, we need to renew those key parameters or change some states.
- If the current interval does not overlap with the active set, we just drop current active set, record some parameters, and create a new active set that contains the current interval.

If this approach doesn’t leave you with enough state information to solve the problem at any given moment, then go for a full fledged sweep line solution.

Problems:

# **Minimum Number of Arrows to Burst Balloons**

[https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/description/](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/description/)

There are some spherical balloons taped onto a flat wall that represents the XY-plane. The balloons are represented as a 2D integer array `points` where `points[i] = [xstart, xend]` denotes a balloon whose **horizontal diameter** stretches between `xstart` and `xend`. You do not know the exact y-coordinates of the balloons.

Arrows can be shot up **directly vertically** (in the positive y-direction) from different points along the x-axis. A balloon with `xstart` and `xend` is **burst** by an arrow shot at `x` if `xstart <= x <= xend`. There is **no limit** to the number of arrows that can be shot. A shot arrow keeps traveling up infinitely, bursting any balloons in its path.

Given the array `points`, return *the **minimum** number of arrows that must be shot to burst all balloons*.

```cpp
class Solution {
public:
int findMinArrowShots(vector<vector<int>>& points) {
        int count = 0, minEnd = INT_MAX;
        sort(points.begin(), points.end());
        for(auto& p: points) {
            if(p[0] > minEnd) {count++; minEnd = p[1];}
            else minEnd = min(minEnd, p[1]);
        }
        return count + !points.empty();
    }
};
```

# **Non-overlapping Intervals**

[https://leetcode.com/problems/non-overlapping-intervals/description/](https://leetcode.com/problems/non-overlapping-intervals/description/)

Given an array of intervals `intervals` where `intervals[i] = [starti, endi]`, return *the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping*.

**Example 1:**

```
Input: intervals = [[1,2],[2,3],[3,4],[1,3]]
Output: 1
Explanation: [1,3] can be removed and the rest of the intervals are non-overlapping.
```

```cpp
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end());
        int minEnd = INT_MIN;
        int count = 0;
        for(auto &interval: intervals) {
            if(interval[0] >= minEnd) {
                minEnd = interval[1];
            }
            else if(interval[0] < minEnd) {
                minEnd = min(minEnd, interval[1]);
                count++;
            }
        }
        return count;
    }
};
```

# **Merge Intervals**

[https://leetcode.com/problems/merge-intervals/description/](https://leetcode.com/problems/merge-intervals/description/)

Given an array of `intervals` where `intervals[i] = [starti, endi]`, merge all overlapping intervals, and return *an array of the non-overlapping intervals that cover all the intervals in the input*.

**Example 1:**

```
Input: intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlap, merge them into [1,6].

```

**Example 2:**

```
Input: intervals = [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considered overlapping.
```