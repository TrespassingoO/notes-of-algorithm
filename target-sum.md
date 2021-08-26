# 题解一：

暴力枚举，二进制法，（《算法竞赛入门经典》第七章 子集生成有对这种方法的讲解）

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int n = nums.size(), cnt = 0;
        for(int i = 0; i < (1<<n); i++) {//每个i代表一种情况（一个表达式），i的二进制形式中1和0分别表示+和-
            int sum = 0;
            for(int j = 0; j < n; j++) {//遍历每一个情况下各位是1还是0，并做相应的操作
                if(i&(1<<j)) sum += nums[j];
                if(!(i&(1<<j))) sum -= nums[j]; 
            }
            if(sum == target) cnt++;
        }
        return cnt;
    }
};
```

时间复杂度O(n*(2^n))，空间复杂度O(1)

上面那样会超时，于是看到[@dianhsu](/u/dianhsu/)的双向搜索的代码，学到了，算是对这种解法的优化。

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
    	int len = nums.size(), half = len/2, sum, ans = 0;
        map<int, int> mp;
        //处理前半段
        for(int i = 0; i < (1<<half); i++) {
            sum = 0;
            for(int j = 0; j < half; j++) {
                if(i&(1<<j)) sum += nums[j];
                else sum -= nums[j];
            }
            mp[sum]++;
        }
        //处理后半段
        for(int i = 0; i < (1<<(len-half)); i++) {//len为奇数时，len-half才为后半段的长度
            sum = 0;
            for(int j = 0; j < len-half; j++) {
                if(i&(1<<j)) sum += nums[j+half];
                else sum -= nums[j+half];
            }
            if(mp.count(target-sum)){
				ans += mp[target-sum];//map的这种用法在leetcode的题1.两数之和中就有用到，还是挺巧妙的。
            }
        }
        return ans;
    }
};
```

由于前一个版本的代码中对没种情况都处理一次其各位，会重复处理前半部分相同的情况(当然不止前半部分)，如++--与++-+，前半部分都是++，而后面这种写法则是分开处理前半段和后半段时间复杂度减半。

时间复杂度：O(n*2^(n/2)), 空间复杂度：O(n/2);

# 解法二：

深度优先搜索(DFS)

