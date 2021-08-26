# [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/) 递归，原地，栈，队列

# 题目：

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

[github题解仓库](https://github.com/TrespassingoO/notes-of-algorithm)

blog：https://blog.csdn.net/github_39329077

# 解法一：

大致流程就是递归到链表的末尾，然后返回反转后的头结点，在返回的过程中反转，并传递回来这个头结点。

抓住reverseList函数返回的是目标链表的头结点这一点思考返回值，思考返回值该用什么接。

引入pre参数，原链表中各个结点head和head->next的关系(head是head->next前驱)在目标链表中会是相反的关系。

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head, ListNode* pre = nullptr) {
        if(!head->next) {
            head->next = pre;
            return head;
        } 
        ListNode* ans = reverseList(head->next, head);
        head->next = pre;
        return ans;
    }
};
```

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head, ListNode* pre = nullptr) {
        if(!head) return pre;
        ListNode* ans = reverseList(head->next, head);
        head->next = pre;
        return ans;
    }
};
```

# 解法二：

另声明两个指针来原地操作

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(!head)   return nullptr;
        ListNode *pre = nullptr, *p;
        while(head->next) {
            p = head->next;
            head ->next = pre;
            pre = head;
            head = p;
        }
        head->next = pre;
        return head;
    }
};
```

# 解法三：

用栈存链表结点的值，再用尾插法直接生成反转后的链表

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode *dummyhead = new ListNode(0);
        dummyhead->next = head;
        ListNode *r;
        stack<int> st;
        while(dummyhead->next != NULL) {
            r = dummyhead->next;
            st.push(r->val);
            dummyhead->next = r->next;
            delete r;
        }
        r = dummyhead;
        while(!st.empty()) {
            ListNode *temp = new ListNode(st.top());
            st.pop();
            temp->next = NULL;
            r->next = temp;
            r = temp;
        }
        return dummyhead->next;
    }
};
```

# 解法四：

用队列存链表结点的值，再用头插法生成反转后的链表

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode *dummyhead = new ListNode(0);
        dummyhead->next = head;
        ListNode *r;
        queue<int> qu;
        while(dummyhead->next != NULL) {
            r = dummyhead->next;
            qu.push(r->val);
            dummyhead->next = r->next;
            delete r;
        }
        while(!qu.empty()) {
            ListNode *temp = new ListNode(qu.front());
            qu.pop();
            temp->next = dummyhead->next;
            dummyhead->next = temp;
        }
        return dummyhead->next;
    }
};
```

