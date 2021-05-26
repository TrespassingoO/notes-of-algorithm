# [897. 递增顺序搜索树](https://leetcode-cn.com/problems/increasing-order-search-tree/)之递归思考

# 题目：

给你一棵二叉搜索树，请你 **按中序遍历** 将其重新排列为一棵递增顺序搜索树，使树中最左边的结点成为树的根结点，并且每个结点没有左子结点，只有一个右子结点。

# 解法一：

## 思路：

函数最终要返回该二叉搜索树最左边的结点，也是以该结点为根结点的递增顺序搜索树，所以设计的递归函数的返回值应该是所给搜索树的最左边的结点`ans`。中序遍历中引入`pos`(后继)这个参数可以很方便的处理前后继之间的关系，`pos`参数表示当前结点在中序遍历中的后继结点。（突然发现这与线索树的建立有异曲同工之妙啊啊啊！（咦，这不就是在考察遍历时前后继之间的关系吗！所以，就得引入后继这个参数！

```c++
class Solution{
public:
	TreeNode* increasingBST(TreeNode* root, TreeNode* pos = nullptr) {
        if(!root) return pos;
        TreeNode* ans = incresaingBST(root->left, root);//当前结点左子树的后继结点就是该结点本身
        root->right = increasingBST(root->right, pos);//pos是中序遍历下当前结点的后继结点
        							//那它是怎么确定的呢？在此之前某次调用root->left时，赋予的root
        root->left = nullptr;
        return ans;
    }   
}
```

## 带你跑：

大体上是左子树进入递归设置好后继结点，当前结点的右指针指向 右子树进入递归调出的后继结点，这两个动作嵌套完成。

在遍历左子树的过程中传入的参数`(root->left, root)`，root为后面的结点确定了的`pos`，也就是其左子树的最右边结点的后继。

当遇到任一结点T的左孩子为空时，结束判断语句就返回该结点T，而第一次遇到的左孩子为空的结点就是原搜索树最左边的结点，所以函数结尾需返回它，在此之前需声明一个`ans`变量保存它。

当遇到任一结点T的右孩子为空时，就直接返回它所在子树的后继或者是最后的`nullptr`。

所以`root->right = increasingBST(root->right,pos)`这段代码既能使当前结点指向其右子树中最左边的结点，还能使其指向祖先结点中的某个后继结点。

总结就是前者铺路，后者搭桥。



# 解法二：

## 思路：

反过来按照右、中、左的顺序递归处理该搜索树（咦！突然发现题目要求中序遍历，但这不算，但这种解法值得一见！

```c++
class Solution{
public:
    void fun(TreeNode* root, TreeNode* ans) {//ans是始终指向拼接中的递增顺序搜索树的根结点的指针。
        if(!root) return;
        fun(root->right, ans)
        root->rigth = ans;
        ans = root;
        fun(root->left, ans)
        root->left = nullptr;
    }
	TreeNode* increasingBST(TreeNode* root){
        TreeNode* ans = nullptr;
        fun(root, ans);
        return tail;
    }
}
```

添加默认参数，合并为一个函数的写法：

递归函数返回递增顺序搜索树的根结点，那么赋值时抓住这点来读代码。(刚开始不太好读明白的。慢慢来（：

```c++
class Solution{
public:
    TreeNode* increasingBST(TreeNode* root, TreeNode* ans = nullptr) {
		if(!root) return ans;//第一次返回nullptr，之后返回传进来的ans(目标搜索树的根结点)。
        ans = increasingBST(root->right, ans);
        root->right = ans;
        ans = root;
        ans = increasingBST(root->left, ans);//这里注意上面的ans要传入到函数中，而被赋值的ans已改变
        root->left = nullptr;//先处理完左分支后才能设为空指针
        return ans;
    }
}
```

进一步精简：

函数功能也可以理解为`root`后面拼接`ans`。

```c++
class Solution{
public:
	TreeNode* increasingBST(TreeNode* root, TreeNode* ans = nullptr) {
        if(!root) return ans;
        root->right = increasingBST(root->right, ans);
        ans = increasing(root->left, root);
        root->left = nullptr;
        return ans;
	}   
}
参考：https://leetcode-cn.com/problems/increasing-order-search-tree/solution/c-yi-ci-bian-li-yuan-di-xiu-gai-by-meng-fzstk/
```

细心读一读还是好理解的。

# 其他：

看到了个有点怪怪的写法：（其实跟解法二一样(效果），但又不太一样(编码思路？)，殊途同归吧

```c++
class Solution {
public:
    TreeNode* solve(TreeNode* root, TreeNode* tail) {
        if (!root) return tail;
        auto rTree = solve(root->right, tail);
        auto lTree = solve(root->left, root);
        root->right = rTree;
       	root->left = NULL;
        return lTree;
    }
    TreeNode* increasingBST(TreeNode* root) {
        return solve(root, NULL);
    }
};
//参考：https://leetcode-cn.com/problems/increasing-order-search-tree/solution/c-di-gui-2chong-jie-fa-fei-di-gui-1chong-r2jv/
```

# 思考：

最后有些对递归的思考，或者说对递归函数参数的思考，写递归函数时，参数大体有两种用法或者说应用场景：

- 一种是以引用或者指针的形式进入函数，在函数的作用下来改变传入参数的值，这样的好处是可以减少全局变量的使用，避免造成变量污染。
- 另一种可以为递归服务（当然也可以是指针或引用的形式），在函数中调用函数本身时可以传入不同含义的参数，达到期望的作用。

以上是对递归函数从应用上概念的划分，在写或者分析递归函数时，除了自上而下思考解决方案，需对各参数有个以上初步的定义和判断，当然这是很基本的，但脑子要清晰。

随后不管是写递归还是分析递归代码，核心就是分析每个参数的含义，有时一个参数会有多个含义，这也是需要注意的，分析参数的含义是真正理解一段递归代码的必要条件，比如树上递归时，该参数在不同结点上的传入浅层有什么含义(比如有时是当前结点的父结点)，或者更本质的含义是什么（比如是遍历的后继结点），或者在终止条件语句中，为什么return的是这个变量而不是其他的，他在不同结点、不同分支上的return的都是什么，函数结尾的return又都返回的是什么。

我觉得分析递归函数的首要原则就是事实求是，函数调用中各参数在位置上的对应关系要分清，各参数的含义要分清。在分清或者试图分清以上两点的基础上去理解递归的含义时，虽然不太容易，但对内功的提升是很有帮助的，同时在纸上或者脑子里跑几一遍，能加深感性上的认识。

反问总结：

从整体递归解决方案上，该参数有什么含义？

从各个结点上对该结点的处理上，该参数有什么含义？

各处return在整体意义上、不同结点、不同分支上都返回的是什么？

这里为什么是xxx,为什么不是其他,还可以用什么代替？

能精简吗，或许有更优雅的写法？

最后，用非递归写法试试？



