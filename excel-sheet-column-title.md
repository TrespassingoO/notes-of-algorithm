# 思路

10进制为例：

4321 = 4 * 10^3 + 3 * 10^2 + 2 * 10^1 + 1 * 10^0 = 10 * (10 * (10 * (10 * 0 + 4) + 3) + 2) + 1



26进制为：

701 = 26*(26*(26*0 + 1) + 0) + 25



# 编码思路

先连除26（除26的过程中 + 后面的余项也再除26 由`int`除法的性质 截断了小数部分），直至边界情况（c <= 26） ，处理掉最里层并赋值给 `ans` ，然后再递归返回的过程中逐层将余项追加到 `ans` 中即可。

需要注意的是'A'代表1，

 

# 代码



```cpp
class Solution {
public:
    string ans;
    void dfs(int c) {
        if(c <= 26) {
            char t = 'A' + c - 1;
            ans = t;
            return;
        }
        dfs((c-1)/26);
        char s = 'A' + (c-1)%26;
        ans += s;
    }
    string convertToTitle(int columnNumber) {
        dfs(columnNumber);
        return ans;
    }
};
```

