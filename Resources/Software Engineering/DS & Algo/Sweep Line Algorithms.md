# Sweep Line Algorithms

For sweep line algorithms

- Order all Points of Interests (PoI) left to right, put them in the same structure.
- Instead of storing all the relevant data regarding PoI, just keep an index along with the coordintate of the PoI that points to a DS that has all this data.
- Keep in mind, multiple PoI can have the same coordinate; you have to process all these PoIs at the same time.
DO NOT RELY ON CLEVERNESS TO GET AROUND THIS. BEST IS TO PROCESS THEM TOGETHER, TO KEEP THINGS PREDICTABLE.

Example, the SkyLine Problem [https://leetcode.com/problems/the-skyline-problem/submissions/1081201657/](https://leetcode.com/problems/the-skyline-problem/submissions/1081201657/)

Here

- PoI: leading or trailing edge of a building
- Coordinate of PoI: x coordinate of PoI
- Index that points to data about building: b

```cpp
class Solution {
public:

    vector<vector<int>> getSkyline(vector<vector<int>>& buildings) {
        vector<vector<int>> edges;
        for(int b=0; b < buildings.size(); b++) {
            edges.push_back({buildings[b][0], b});
            edges.push_back({buildings[b][1], b});
        }
        sort(edges.begin(), edges.end());

        // will hold {height, trailing edge}
        auto comp = [](const vector<int> &x, const vector<int> &y) -> bool {return x[0] < y[0];};
        priority_queue <vector<int>, vector<vector<int>>, decltype(comp)> PQ(comp);

        int height = 0;
        vector<vector<int>> ans;
        for(int e=0; e<edges.size();) {
            int x = edges[e][0];
            while(!PQ.empty() && (PQ.top())[1] <= x)
                PQ.pop();
            while(e < edges.size() && edges[e][0] == x) {
                int b = edges[e][1];
                // if leading edge
                if(buildings[b][0] == x) {
                    PQ.push({buildings[b][2], buildings[b][1]});
                }
                e++;
            }
            int currentHeight = 0;
            if(!PQ.empty())
                currentHeight = (PQ.top())[0];
            if(height != currentHeight) {
                height = currentHeight;
                ans.push_back({x, height});
            }
        }

        return ans;
    }
};
```