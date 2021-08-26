# [523. 连续的子数组和](https://leetcode-cn.com/problems/continuous-subarray-sum/) 同余定理，前缀和

# 题目：

给你一个下标从 0 开始的二进制字符串 s 和两个整数 minJump 和 maxJump 。一开始，你在下标 0 处，且该位置的值一定为 '0' 。当同时满足如下条件时，你可以从下标 i 移动到下标 j 处：

i + minJump <= j <= min(i + maxJump, s.length - 1) 且
s[j] == '0'.
如果你可以到达 s 的下标 s.length - 1 处，请你返回 true ，否则返回 false 。

# 解法一：

区间和：

由前缀和的性质，`preSum[j] - preSum[i]`可以得到 (i+1) ~ j 区间内的和。

余数等于k：

作判断时需用到同余定理，不然会超时。

同余定理总结下：

- (a+b)mod n = ((a mod n)+(b mod n)) mod n

- ((a-b) mod n = ((a mod n) - (b mod n)) mod n

- ab mod n = ((a mod n) (b mod n) mod n


- **(a - b)mod n = 0 -> a mod n = b mod n**

于是我们可以用上最后一条的性质，从`i = 2` 开始循环，每次插入`preSum[i-2]%k`到容器中，并对`preSum[i]%k`作判断，天然的能保证`preSum[i]%k`是与`i-2`及之前的前缀和作判断即：

`((preSum[i] - preSum[j]) mod k = 0 -> preSum[i] mod k == preSum[j] mod k == 0或t, j = 0~i-2 )`，这样就可以不用map存键值对，直接用set存`preSum`就好。同时我们还可以边求前缀和边作判断。

```c++
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        const int len = nums.size();
        int preSum[len+2];
        preSum[0] = 0, preSum[1] = nums[0]%k;
        for(int i = 2; i <= len; i++) {
            preSum[i] = (preSum[i-1] + nums[i-1])%k;
            set.insert(preSum[i-2]);
            if(set.count(preSum[i]%k))
                return true;
        }
        return false;
    }
private:
    unordered_set<int> set;
};
```

# 解法二：

暴力解法，感觉还是挺简练的，虽然会超时：

```c++
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        const int len = nums.size();
        int preSum[len+2];
        preSum[0] = 0;
        for(int i = 1; i <= len; i++)
            preSum[i] = (preSum[i-1] + nums[i-1])%k;
        for(int i = 0, j = len; j != 2 ; i++) {
            if(i == j-1 && i) j--, i = 0;//有j = 2的情况。
            if((preSum[j]-preSum[i])%k == 0)
                return true;
        }
        return false;
    }
};
```

