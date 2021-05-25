# [993. 二叉树的堂兄弟节点](https://leetcode-cn.com/problems/cousins-in-binary-tree/) 之妙不可言的位操作

# 题目：

在二叉树中，根节点位于深度 0 处，每个深度为 k 的节点的子节点位于深度 k+1 处。

如果二叉树的两个节点深度相同，但 父节点不同 ，则它们是一对堂兄弟节点。

我们给出了具有唯一值的二叉树的根节点 root ，以及树中两个不同节点的值 x 和 y 。

只有与值 x 和 y 对应的节点是堂兄弟节点时，才返回 true 。否则，返回 false。

来源：力扣（LeetCode）

[github题解仓库](题解链接：https://github.com/TrespassingoO/notes-of-algorithm)

blog：https://blog.csdn.net/github_39329077

# 解法一：

判断是否为堂兄弟：深度相同，父节点不同。所以要有`depth`参数和`parent`参数，fun函数作先序遍历时找到target（x or y）就以pair的形式返回深度和父节点的值（因为二叉树中各节点值唯一所以可以仅用父节点的val来判断父节点是否相同），`x`、`y`进入`fun`找到自己的位置后直接返回结果并作剪枝操作，避免浪费时间作无用的搜索。

即：DFS+pair+剪枝

```cpp
class Solution {
public:
    bool isCousins(TreeNode* root, int x, int y) {
        pair<int, int> p_x = fun(root, nullptr, 0, x);
        pair<int, int> p_y = fun(root, nullptr, 0, y);
        if(p_x.first == p_y.first && p_x.second != p_y.second)
            return true;
        else
            return false;
    }
    pair<int, int> fun(TreeNode* root, TreeNode* parent, int depth, int target) {
        if(!root)
            return make_pair(-1, -1);
        if(root->val == target) {
            return make_pair(depth, parent == nullptr ? -1: parent->val);//需注意下parent为根节点时
        }
        pair<int, int> p = fun(root->left, root, depth+1, target);
        if(p.first == -1 && p.second == -1)
            p = fun(root->right, root, depth+1, target);
        return p;
    }
};
```

从@guaidaokakaxi那学到的对位操作的小技巧，只要数据范围小于2^16就可以同时将两个数放入一个32位的int型变量里。（这题数据比较小都够放4个数了

即：DFS+int位操作，确实优雅些

```cpp
class Solution {
public:
    bool isCousins(TreeNode* root, int x, int y) {
        int temp_x = fun(root, nullptr, 0, x);
        int temp_y = fun(root, nullptr, 0, y);
        int depth_x = temp_x >> 16;
        int depth_y = temp_y >> 16;
        if(depth_x != depth_y)
            return false;
        int parent_x_val = temp_x & 0x0000ffff;
        int parent_y_val = temp_y & 0x0000ffff;
        if(parent_x_val != parent_y_val)
            return true;
        else
            return false;
    }
    int fun(TreeNode* root, TreeNode* parent, int depth, int target) {
        if(!root)
            return 0;
        if(root->val == target) {
            return (depth << 16) + (parent == nullptr ? 0 : parent->val);
        }
        int temp = fun(root->left, root, depth+1, target);
        if(temp == 0)
            temp = fun(root->right, root, depth+1, target);
        return temp;
    }
};
```

# 解法二：

这个解法从@Edward Elric那学到的，妙不可言！类似堆排序时，用数组建堆，这里我们只用数组存树，不作调整，index为节点序号也是节点在数组中的下标(从1开始)，自上而下，从左至右的分配序号，遇到空结点就跳过这个序号（各结点位置的序号永远与一个完全二叉树的序号一一对应，只是本题的树可以不完全）。

那么对任一序号为k的节点，其父节点的序号就是k/2，其左孩子就是k * 2，右孩子就是k * 2+1，同时最精彩的地方来了！ 对于树中**同一层**的节点，其序号(index)二进制下的最高比特位是相同的，如序号分别为10[1010]，12[1100]的节点都在第4层，他们最高2次幂都是8[1000]。

所以就有：

1.通过对下标值除2来判断父节点。

2.通过判断最高比特位是否相同来判断是否属于同一层。

判断最高位的比特是否相同，Edw用的是java里的`toBinaryString(x).lentght()`来实现的，而c++里没有这个方法所以我想到了用异或操作，两个数相异或时，若最高位均为1，1^1 = 0, 则这两个数中的任一个数一定大于异或后的结果，否则总有一个数小于这个结果。（完美

最后为节省空间可以换成`map<int, int>`存树

即：数组存树+序号特性判断

```cpp
class Solution {
public:
    bool isCousins(TreeNode* root, int x, int y) {
        buildTree(root, 1);
        if(mp.find(x) == mp.end() || mp.find(y) == mp.end())
            return false;
        int num_x = mp.find(x)->second;
        int num_y = mp.find(y)->second;
        if(num_x/2 != num_y/2 && (num_x>(num_x^num_y) && num_y>(num_x^num_y)))
            return true;
        else
            return false;
    }
    void buildTree(TreeNode* root, int index) {
        if(!root)
            return;
        mp[root->val] = index;
        buildTree(root->left, index<<1);
        buildTree(root->right, (index<<1)+1);
    }
private:
    unordered_map<int, int> mp; 
};
```

既然我们只x，y在树上的序号，数组也可以不要，遍历的过程中存下他们的下标最后返回即可，同时还可以用到上面学到的int存两数的小技巧。

称之：一”位“到底：

```cpp
class Solution {
public:
    bool isCousins(TreeNode* root, int x, int y) {
        int temp = 0;
        buildTree(root, 1, x, y, temp);
        if(!temp || !(temp&0xffff0000) || !(temp&0x0000ffff))
            return false;
        int num_x = temp>>16;
        int num_y = temp&0x0000ffff;
        if(num_x/2 != num_y/2 && (num_x>(num_x^num_y) && num_y>(num_x^num_y)))
            return true;
        else
            return false;
    }
    int buildTree(TreeNode* root, int index, int target_x, int target_y, int &temp) {
        if(!root)
            return temp;
        if(root->val == target_x)//各节点的值唯一所以不会重复加
            temp += index<<16;
        else if(root->val == target_y)
            temp += index;
        if(temp&0xffff0000 && temp&0x0000ffff)//必要的剪枝
            return temp;
        temp = buildTree(root->left, index<<1, target_x, target_y, temp);
        if(!(temp&0xffff0000) || !(temp&0x0000ffff))//必要的剪枝
            temp = buildTree(root->right, (index<<1)+1, target_x, target_y, temp);
        return temp;
};
```

temp太多还是换种写法：

```cpp
class Solution {
public:
    bool isCousins(TreeNode* root, int x, int y) {
        int temp = 0;
        buildTree(root, 1, x, y, temp);
        if(!temp || !(temp&0xffff0000) || !(temp&0x0000ffff))
            return false;
        int num_x = temp>>16;
        int num_y = temp&0x0000ffff;
        if(num_x/2 != num_y/2 && (num_x>(num_x^num_y) && num_y>(num_x^num_y)))
            return true;
        else
            return false;
    }
    void buildTree(TreeNode* root, int index, int target_x, int target_y, int &temp) {
        if(!root)
            return ;
        if(root->val == target_x)//各节点的值唯一所以不会重复加
            temp += index<<16;
            else if(root->val == target_y)
                temp += index;
        if(temp&0xffff0000 && temp&0x0000ffff)
            return ;
        buildTree(root->left, index<<1, target_x, target_y, temp);
        if(!(temp&0xffff0000) || !(temp&0x0000ffff))
            buildTree(root->right, (index<<1)+1, target_x, target_y, temp);
        return ;
    }
};
```



# 解法三：

BFS

用`preDepth`来判断是否还在同一层，用`comParent`来判断当前节点的左右孩子是否就是所要找的x，y。

```cpp
class Solution {
public:
    bool isCousins(TreeNode* root, int x, int y) {
        queue<pair<TreeNode*, int> > qu;
        int curDepth =1, preDepth = 1;
        bool find_x = false, find_y = false, comParent = false;
        qu.push(make_pair(root, curDepth));
        if(root->val == x || root-> val == y)
            return false;
        while(!qu.empty()) {
            auto [cur,curDepth] = qu.front();
            if(preDepth != curDepth && (find_x || find_y))//若遍历到下一层，仅找到一个数，直接不符合
                return false;
            preDepth = curDepth;//将当前节点的深度赋给preDepth用于对下个节点判断是否换层了
            qu.pop();
            if(cur->left) {
                qu.push(make_pair(cur->left, curDepth+1));
                if(cur->left->val == x) find_x = true, comParent = true;
                    else if(cur->left->val == y) find_y = true, comParent = true;
            }
            if(find_x && find_y)
                return true;
            if(cur->right) {
                qu.push(make_pair(cur->right, curDepth+1));
                if(cur->right->val == x) find_x = true;
                    else if(cur->right->val == y) find_y = true;
            }
            if(find_x && find_y)
            	return comParent ? false : true;
            comParent = false;
        }
        return false;
    }
};
```



看了@Qian的写法，太美了，贴来膜拜下

每次直接将一层的节点放入队列，并存下找到的x，y的父节点，然后一并处理。

```cpp
class Solution {
public:
    using PTT = pair<TreeNode*, TreeNode*>;
    bool isCousins(TreeNode* root, int x, int y) {
        // 使用队列q来进行bfs
        // 其中pair中，p.first 记录当前结点的指针，p.second 记录当前结点的父结点的指针
        queue<PTT> q;
        q.push({root, nullptr});
        while(!q.empty()) {
            int n = q.size();
            vector<TreeNode*> rec_parent;
            for(int i = 0; i < n; i++) {
                auto [cur, parent] = q.front(); q.pop();
                if(cur->val == x || cur->val == y)
                    rec_parent.push_back(parent);
                if(cur->left) q.push({cur->left, cur});
                if(cur->right) q.push({cur->right, cur});
            }
            // `x` 和 `y` 都没出现
            if(rec_parent.size() == 0)
                continue;
            // `x` 和 `y` 只出现一个
            else if(rec_parent.size() == 1)
                return false;
            // `x` 和 `y` 都出现了
            else if(rec_parent.size() == 2)
                // `x` 和 `y` 父节点 相同/不相同 ？
                return rec_parent[0] != rec_parent[1];
        }
        return false;
    }
};

作者：QianK
链接：https://leetcode-cn.com/problems/cousins-in-binary-tree/solution/c-you-ya-bfs-by-qiank-9vk9/
来源：力扣（LeetCode）
```

# 小技巧：

- 数组存树时，下标从1开始，各层节点序号的最高比特位相同。用`(x > (x^y) && y > (x^y))`来判断是否同层
- `int`作位操作，存多个数
- 善用容器的size()函数

