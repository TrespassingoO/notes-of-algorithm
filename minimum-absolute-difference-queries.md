@[int65536](https://leetcode-cn.com/u/int65536/)

```cpp
//c[v] v表示 query数组中查询区间右端点为v 的下标的集合
//遍历nums数组，i为下标，维护last的同时，遍历 查询区间右端点为i的 c[i]s
//维护一个last数组，记录每个nums[i]最后一次出现的位置为i
//当num最后一次出现的下标在查询区间内时，表示区间中有这个值，可以作相减操作

class Solution
{
public:
    vector<int> minDifference(vector<int>& a, vector<vector<int>>& q)
    {
        int n = a.size();
        int m = q.size();
        vector<int> ret(m);
        vector<vector<int>> c(n);
        for (int i = 0; i < m; ++i) {
            c[q[i][1]].push_back(i);
        }
        const int M = 100;
        for (int i = 0; i < n; ++i) {
            --a[i];
        }
        vector<int> last(M, -1);
        for (int i = 0; i < n; ++i) {
            last[a[i]] = i;// 表示 值为nums[i] 最后一次出现的下标 
            for (int id : c[i]) {//c[i] query中 查询区间右端点为i 的每个下标id
                int s = q[id][0];
                ret[id] = M + 1;
                int prev = -1;
                for (int j = 0; j < M; ++j) {
                    if (last[j] >= s) {//last[j], j离i越近越有可能在q[id][0]~q[id][1]区间内
                        if (prev >= 0) {
                            ret[id] = min(ret[id], j - prev);
                        }
                        prev = j;
                    }
                }
                if (ret[id] > M) {
                    ret[id] = -1;
                }
            }
        }
        return ret;
    }
};
```



```cpp
const int MAXN = 1e5+5;
const int INF = 1e4;
int presum[MAXN][102];//[i][v] 表示nums前i位中，v出现的次数
class Solution {
public:
    vector<int> minDifference(vector<int>& nums, vector<vector<int>>& Q) {
        int n = nums.size();
        for(int i = 0; i < n; i++) for(int v = 0; v <= 100; v++) presum[i][v] = 0;
        
        //前缀和初始化，前继承上一层，然后给出现的位置值加一
        for(int i = 1; i <= n; i++) {
            for(int v = 0; v <= 100; v++) presum[i][v] = presum[i-1][v];
            presum[i][nums[i-1]]++;
        }

        vector<int> ans;
        for(int i = 0; i < Q.size(); i++) {
            int l = Q[i][0]+1, r = Q[i][1]+1;
            int last = INF, cur = INF;
            for(int v = 100; v > 0; v--) {
                if(presum[r][v] - presum[l-1][v] == 0) continue;
                if(last != INF) cur = min(cur, last - v);
                if(cur == 1) break;
                last = v;
            }
            if(cur == INF) cur = -1;
            ans.push_back(cur);
        }
        return ans;
    }
};
```

