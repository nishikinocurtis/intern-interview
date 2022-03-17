---
title: 剑指Offer 汇总
tags:
  - 算法
  - cpp
categories:
  - 剑指Offer
abbrlink: 7f86
date: 2022-03-17 19:20:46
---



剑指Offer 汇总

<!--more-->

## 链表

### [剑指 Offer 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)



用栈倒一下顺序

**代码**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
    stack<int> stack;
public:
    vector<int> reversePrint(ListNode *head) {
        vector<int> ans;
        ListNode *p = head;
        if (!p) {
            return ans;
        }
        while (p) {
            stack.push(p->val);
            p = p->next;
        }
        while (!stack.empty()) {
            ans.push_back(stack.top());
            stack.pop();
        }
        return ans;
    }
};
```



### [剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)



运用迭代，储存当前节点的next和prev。

prev用于反转链表，next用于将当前指针向下迭代。

每一次迭代，反转p和prev之间的指针。

由于头指针需要指向NULL，所以我们一开始将prev设为NULL，将p指向头指针。

**代码**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *reverseList(ListNode *head) {
        ListNode *p = head, *prev = NULL, *next;
        while (p) {
            next = p->next;
            p->next = prev;
            prev = p;
            p = next;
        }
        return prev;
    }
};
```



### [剑指 Offer 35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)



我们使用递归求解此题，每当创建一个节点时，递归创建该节点的next和random节点。

我们用哈希表记录每一个节点对应新节点的创建情况。

我们需要首先检查当前节点是否被拷贝过，如果已经拷贝过，我们可以直接从哈希表中取出拷贝后的节点的指针并返回。

**代码**

```c++
/*
// Definition for a Node.
class Node {
public:  
    int val;
    Node* next;
    Node* random;
    
    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/
class Solution {
    map<Node *, Node *> map;
public:
    Node *copyRandomList(Node *head) {
        if (!head) {
            return NULL;
        }
        if (!map.count(head)) {
            //如果节点不存在，创建节点并加到map中
            Node *node = new Node(head->val);
            map[head] = node;
            //递归创建next
            node->next = copyRandomList(head->next);
            //递归创建random
            node->random = copyRandomList(head->random);
        }
        return map[head];
    }
};
```



## 排序

### [剑指 Offer 45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)



我们只关注排序规则，因为每个排序算法最后的顺序，只涉及到最基础两个元素之间的比较。

我们可以自定义 C++ sort 函数的排序规则。

**代码**

```c++
class Solution {
public:
    string minNumber(vector<int> &nums) {
        vector<string> strings;
        string ans;
        for (int i = 0; i < nums.size(); i++) {
            strings.push_back(to_string(nums[i]));
        }
        sort(strings.begin(), strings.end(), cmp);
        for (int i = 0; i < strings.size(); i++) {
            ans.append(strings[i]);
        }
        return ans;
    }

    static bool cmp(string &str1, string &str2) {
        return str1 + str2 < str2 + str1;
    }
};
```



### [剑指 Offer 61. 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)



需满足 `max - min < 5` 。

我们先排序，统计 joker 牌的数目。

**代码**

```c++
class Solution {
public:
    bool isStraight(vector<int> &nums) {
        sort(nums.begin(), nums.end());
        int joker = 0;
        for (int i = 0; i < nums.size() - 1; ++i) {
            if (nums[i] == 0)
                joker++;
            //确保不重复，才能成顺子
            if (nums[i] != 0 && nums[i] == nums[i + 1])
                return false;
        }
        if (nums[4] - nums[joker] < 5)
            return true;
        else
            return false;
    }
};
```



### [剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)



**方法一**

大根堆，大根堆的顶部总是最大的值，如果新的值比堆顶的值大，则新值入堆，堆顶值出堆。

C++ 使用 priority_queue 来作为大根堆。

**代码**

```c++
class Solution {
public:
    vector<int> getLeastNumbers(vector<int> &arr, int k) {
        vector<int> ans;
        if (k == 0)
            return ans;
        //priority_queue 即为大根堆
        priority_queue<int> queue;
        for (int i = 0; i < k; ++i) {
            queue.push(arr[i]);
        }
        for (int i = k; i < arr.size(); ++i) {
            if (arr[i] < queue.top()) {
                queue.pop();
                queue.push(arr[i]);
            }
        }
        for (int i = 0; i < k; ++i) {
            ans.push_back(queue.top());
            queue.pop();
        }
        return ans;
    }
};
```



**方法二**

快速选择，基于快速排序的思想。

参考 [排序算法 - Allen Ji's blog](https://blog.allenji.cn/p/ef35/) 中的快速排序，我们不需要对分割函数做出修改，只需要修改快速选择函数即可。

每次分割后我们能计算出 pivot 左边分割好的数组长度 num，将其与 k 比较。

- `num == k`：直接返回。
- `k < num`：需要的 k 比分割好的 num 小，说明分割的粒度太粗，重新往小了分割
- `k < num`：需要的 k 比分割好的 num 大，说明分割的不够，重新往大了分割



**代码**

```c++
class Solution {
public:
    vector<int> getLeastNumbers(vector<int> &arr, int k) {
        vector<int> ans;
        if (k == 0 || arr.size() == 0)
            return ans;
        quickSelect(arr, 0, arr.size() - 1, k);
        for (int i = 0; i < k; ++i) {
            ans.push_back(arr[i]);
        }
        return ans;
    }

    //快速选择，遇到返回的位置
    void quickSelect(vector<int> &nums, int left, int right, int k) {
        if (left >= right)
            return;
        //idx 为分割后的 pivot 位置
        int idx = partition(nums, left, right);
        //len 为刚刚一趟分割后 pivot 左边的序列长度
        int len = idx - left + 1;
        if (len == k) {
            return;
        } else if (k < len)
            //需要的比分割好的小，说明分割的粒度太粗，重新往小了分割
            return quickSelect(nums, left, idx - 1, k);
        else
            //需要的比分割好的大，说明分割的不够，重新往大了分割
            return quickSelect(nums, idx + 1, right, k - len);
    }

    //随机锚点分割函数，返回 pivot 在数组中的位置
    int partition(vector<int> &nums, int left, int right) {
        //随机锚点
        int randomIndex = left + 1 + rand() % (right - left);
        swap(nums[left], nums[randomIndex]);

        int pivot = left, slow = pivot + 1;
        for (int fast = slow; fast <= right; fast++) {
            if (nums[fast] < nums[pivot]) {
                swap(nums[fast], nums[slow]);
                slow++;
            }
        }
        swap(nums[pivot], nums[slow - 1]);
        return slow - 1;
    }
};
```



### [剑指 Offer 41. 数据流中的中位数](https://leetcode-cn.com/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/)



本题使用两个堆，一个大顶堆 maxHeap 储存较小的元素，小顶堆 minHeap 储存较大的元素。

这样可以让我们随时访问较大的一半数中最小的元素，较小的一半数中最大的元素，然后进行中位数的计算。

**注意**

- 当两个堆中元素数相同时，向大顶堆添加元素，不同时，向小顶堆添加元素
- 向大顶堆添加时，先向小顶堆中添加，拿到小顶堆中最小的元素，再将其添加到大顶堆
- 向小顶堆中添加同理

**代码**

```c++
class MedianFinder {
    //左边的大顶堆
    priority_queue<int, vector<int>, less<int>> maxHeap;
    //右边的小顶堆
    priority_queue<int, vector<int>, greater<int>> minHeap;
public:
    /** initialize your data structure here. */
    MedianFinder() {
        while (!maxHeap.empty())
            maxHeap.pop();
        while (!minHeap.empty())
            minHeap.pop();
    }

    void addNum(int num) {
        if (maxHeap.size() == minHeap.size()) {
            //大小相等，优先向大顶堆添加
            //具体操作时先添加到小顶堆，拿到小顶堆中最小的，再添加到大顶堆中
            minHeap.push(num);
            maxHeap.push(minHeap.top());
            minHeap.pop();
        } else {
            //同理
            maxHeap.push(num);
            minHeap.push(maxHeap.top());
            maxHeap.pop();
        }

    }

    double findMedian() {
        if (maxHeap.size() == minHeap.size()) {
            //大小相等，返回均值
            return ((double) maxHeap.top() + minHeap.top()) / 2;
        } else
            //大顶堆元素多，返回大顶堆的堆顶
            return (double) maxHeap.top();
    }
};

/**
 * Your MedianFinder object will be instantiated and called as such:
 * MedianFinder* obj = new MedianFinder();
 * obj->addNum(num);
 * double param_2 = obj->findMedian();
 */
```



## 双指针

### [剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)



双指针遍历。

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *deleteNode(ListNode *head, int val) {
        //虚拟头节点，避免多余的判断
        ListNode *pre, *dummyNode = new ListNode(-1), *p = head;
        dummyNode->next = head;
        pre = dummyNode;
        while (p->val != val) {
            pre = p;
            p = p->next;
        }
        pre->next = p->next;
        return dummyNode->next;
    }
};
```



### [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)



快慢指针。

快指针先于k个位置出发，然后两个指针同步走。

快指针走到尾时，慢指针离尾有k的距离。

**代码**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getKthFromEnd(ListNode *head, int k) {
        ListNode *fast = head, *slow = head;
        for (int i = 0; i < k; i++) {
            fast = fast->next;
        }
        while (fast) {
            fast = fast->next;
            slow = slow->next;
        }
        return slow;
    }
};
```



### [剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)



合并两个链表。

- 可使用头节点 head，方便操作。
- 使用 tail 指向链表的尾，tail->next 即是下次要添加的位置。

**代码**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
        ListNode *head = new ListNode(0), *tail = head;
        ListNode *aPtr = l1, *bPtr = l2;
        while (aPtr && bPtr) {
            if (aPtr->val < bPtr->val) {
                tail->next = aPtr;
                aPtr = aPtr->next;
            } else {
                tail->next = bPtr;
                bPtr = bPtr->next;
            }
            tail = tail->next;
        }
        //将未遍历完的链表直接添加到 tail 后
        tail->next = aPtr ? aPtr : bPtr;
        return head->next;
    }
};
```





### [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

本题运用双指针。

> 怎么让两个速度相同，跑道不同的人相遇？
>
> 答案是交换他们的跑道

我们让两个指针分别从两条链表的头开始向后走，当其中一个链表走到尾端（即指向为空）时，让其从另一个链表头开始再走一遍，直到俩链表相遇或是都为空。

假设A链表长为A，B链表长为B，公共区域长为C。

从A出发的指针pA走完时，长度为A+C，相对的，pB走完时长度为B+C。

这时交换他们的跑道，pA走A+C+B，pB走B+C+A，因他们速度相同，要么在交点相遇，要么都走到空节点，此时返回他们指向的节点即可。



**代码**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode *aPtr = headA, *bPtr = headB;
        while (aPtr && bPtr) {
            //如果相同，则找到第一个公共节点
            if (aPtr == bPtr)
                return aPtr;
            aPtr = aPtr->next;
            bPtr = bPtr->next;
            //如果都为空，说明无公共节点
            if (!aPtr && !bPtr)
                return NULL;
            //两个指针走到尽头时，分别切换到另一个链表上
            if (!aPtr)
                aPtr = headB;
            if (!bPtr)
                bPtr = headA;
        }
        return NULL;
    }
};
```



### [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)



头尾指针，头指针只选择偶数，尾指针只选择奇数。

```c++
class Solution {
public:
    vector<int> exchange(vector<int> &nums) {
        int left = 0, right = nums.size() - 1;
        while (left < right) {
            while (left < right && nums[left] % 2 != 0)
                left++;
            while (left < right && nums[right] % 2 == 0)
                right--;
            swap(nums[left], nums[right]);
        }
        return nums;
    }
};
```



### [剑指 Offer 57. 和为s的两个数字](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/)



头尾指针。

双指针之和大于 target ，尾指针减小，使和更小。

小于 target ，头指针增大，使和更大。

```c++
class Solution {
public:
    vector<int> twoSum(vector<int> &nums, int target) {
        int left = 0, right = nums.size() - 1;
        vector<int> ans;
        while (left < right) {
            int sum = nums[left] + nums[right];
            if (sum < target)
                left++;
            else if (sum > target)
                right--;
            else
                break;
        }
        ans.push_back(nums[left]);
        ans.push_back(nums[right]);
        return ans;
    }
};
```



### [剑指 Offer 58 - I. 翻转单词顺序](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)



倒序的快慢指针。

使用快慢指针搜索每一个单词，注意边界。

**代码**

```c++
class Solution {
public:
    string reverseWords(string s) {
        int fast = s.size() - 1, slow;
        vector<string> vector;
        string ans;
        if (s.empty())
            return ans;
        //初始化到第一个非空字符
        while (fast >= 0 && s[fast] == ' ') {
            fast--;
        }
        slow = fast;
        //每次一个循环遍历一个单词加空隙
        while (fast >= 0) {
            //fast 指针遍历完整单词
            while (fast >= 0 && s[fast] != ' ')
                fast--;
            //将单词添加到数组
            vector.push_back(s.substr(fast + 1, slow - fast));
            //跳过空格
            while (fast >= 0 && s[fast] == ' ')
                fast--;
            slow = fast;
        }
        //拼接字符串
        for (int i = 0; i < vector.size(); ++i) {
            if (i != 0)
                ans += ' ';
            ans += vector[i];
        }
        return ans;
    }
};
```



## 搜索与回溯算法

### [面试题32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)



最普通的层序遍历，使用队列。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
    queue<TreeNode *> queue;
    vector<int> ans;
public:
    vector<int> levelOrder(TreeNode *root) {
        if (!root)
            return ans;
        queue.push(root);
        while (!queue.empty()) {
            ans.push_back(queue.front()->val);
            if (queue.front()->left)
                queue.push(queue.front()->left);
            if (queue.front()->right)
                queue.push(queue.front()->right);
            queue.pop();
        }
        return ans;
    }
};
```



### [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)



层序遍历我们一般使用队列，从左向右遍历时，将下一层的节点加入队列。

此题需要我们进行分层，我们便在while语句中进一步用for来细化各层，将开启每一层时队列的大小当作循环条件。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
    queue<TreeNode *> queue;
    vector<vector<int>> ans;
public:
    vector<vector<int>> levelOrder(TreeNode *root) {
        if (!root)
            return ans;
        queue.push(root);
        while (!queue.empty()) {
            int levelSize = queue.size();
            ans.push_back(vector<int>());
            for (int i = 0; i < levelSize; ++i) {
                ans.back().push_back(queue.front()->val);
                if (queue.front()->left)
                    queue.push(queue.front()->left);
                if (queue.front()->right)
                    queue.push(queue.front()->right);
                queue.pop();
            }
        }
        return ans;
    }
};
```



### [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)



我们将奇数层和偶数层的操作分开。

level从0开始计数。

- 奇数层：队首读，队尾入。
- 偶数层：队尾读，队首入。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
    deque<TreeNode *> deque;
    vector<vector<int>> ans;
public:
    vector<vector<int>> levelOrder(TreeNode *root) {
        if (!root)
            return ans;
        deque.push_back(root);
        int level = 0;
        while (!deque.empty()) {
            int levelSize = deque.size();
            ans.push_back(vector<int>());
            for (int i = 0; i < levelSize; ++i) {
                if (level % 2 == 0) {
                    //偶数层，从队尾读，队首入
                    ans.back().push_back(deque.back()->val);
                    if (deque.back()->left)
                        deque.push_front(deque.back()->left);
                    if (deque.back()->right)
                        deque.push_front(deque.back()->right);
                    deque.pop_back();
                } else {
                    //奇数层，队首读，队尾入
                    ans.back().push_back(deque.front()->val);
                    if (deque.front()->right)
                        deque.push_back(deque.front()->right);
                    if (deque.front()->left)
                        deque.push_back(deque.front()->left);
                    deque.pop_front();
                }
            }
            level++;
        }
        return ans;
    }
};
```







### [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)



我们先找到A中与B的根节点数值相同的节点。

然后再用递归比对B是否为A的子结构。

**策略**

- 判断当前节点值是否相等，不等则返回 false，相等则递归继续判断。
- 如果B先递归完，说明之前的判断都没有问题，B为A的子结构，返回 true。
- 如果A先递归完，说明B的结构比A的长，B不为A的子结构，返回 false。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    //遍历 A 的节点
    bool isSubStructure(TreeNode *A, TreeNode *B) {
        if (!A || !B)
            return false;
        //判断是否相等并且为子结构
        if (A->val == B->val && recur(A, B))
            return true;
        //递归搜索
        return isSubStructure(A->left, B) || isSubStructure(A->right, B);
    }

    //检查两棵树是否出现子结构
    bool recur(TreeNode *A, TreeNode *B) {
        //如果 B 先走完，说明 A 包含 B
        if (!B)
            return true;
        //如果 A 先走完，说明 A 不包含 B
        if (!A)
            return false;
        //如果在 B 走完之前出现节点值不相等，说明不包含子结构
        if (A->val != B->val)
            return false;
        //递归继续判断
        return recur(A->left, B->left) && recur(A->right, B->right);
    }
};
```



### [剑指 Offer 27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)



本体应用较基础的递归思想，直接递归交换即可。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode *mirrorTree(TreeNode *root) {
        swapNode(root);
        return root;
    }

    //递归交换
    void swapNode(TreeNode *root) {
        if (!root)
            return;
        TreeNode *temp = root->left;
        root->left = root->right;
        root->right = temp;
        swapNode(root->left);
        swapNode(root->right);
    }
};
```



### [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)



当有两个对称节点，他们应满足以下条件：

- 两个节点值相同
- 左节点的右子节点与右节点的左子节点相同
- 左节点的左子节点与右节点的右子节点相同

我们可以每次递归判断第一个条件，其他的则由递归来完成。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    bool isSymmetric(TreeNode *root) {
        if (!root)
            return true;
        return recue(root->left, root->right);
    }

    bool recue(TreeNode *left, TreeNode *right) {
        //两节点都为空，对称
        if (!left && !right)
            return true;
        //其中一个为空，不对称
        if (left && !right || !left && right)
            return false;
        //两节点值相等，递归判断子节点
        if (left->val == right->val)
            return recue(left->left, right->right) && recue(left->right, right->left);
        return false;
    }
};
```



### [剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)



给定一个 m x n 二维字符网格 board 和一个字符串单词 word 。如果 word 存在于网格中，返回 true ；否则，返回 false 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

  **题解**

DFS + 回溯。

本题 DFS 返回值为是否搜索到。

通过把访问过的 board 元素改为 `#` 。在原地实现了 isVisited 矩阵的功能，无需开辟新空间。

回溯时需要将 board 恢复，避免影响别的层搜索。

**代码**

```c++
class Solution {
    int rows, cols;
    string str;
public:
    bool exist(vector<vector<char>> &board, string word) {
        rows = board.size();
        cols = board[0].size();
        str = word;
        //每一位都要当作起点搜一遍
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (dfs(board, i, j, 0))
                    return true;
            }
        }
        return false;
    }

    bool dfs(vector<vector<char>> &board, int row, int col, int depth) {
        //越界或已经搜索或当前字符不匹配
        if (row >= rows || row < 0 || col >= cols || col < 0 || board[row][col] != str[depth])
            return false;
        //搜索到最后一层，返回
        if (depth == str.size() - 1)
            return true;
        board[row][col] = '#';
        bool res = dfs(board, row + 1, col, depth + 1) || dfs(board, row - 1, col, depth + 1) ||
                   dfs(board, row, col + 1, depth + 1) || dfs(board, row, col - 1, depth + 1);
        //回溯，避免影响别的层的搜索
        board[row][col] = str[depth];
        return res;
    }
};
```



### [剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)



地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

 **题解**

DFS + 回溯。

本题 DFS 返回值为当前可到达的格子数。

**代码**

```c++
class Solution {
    int rows, cols, kk;
    vector<vector<bool>> isVisited;
public:
    int movingCount(int m, int n, int k) {
        isVisited = vector<vector<bool>>(m, vector<bool>(n));
        rows = m;
        cols = n;
        kk = k;
        return dfs(0, 0);
    }

    int dfs(int row, int col) {
        //越界或是否已访问或不满足条件
        if (row > rows - 1 || col > cols - 1 || isVisited[row][col] || kk < bitSum(row) + bitSum(col))
            return 0;
        isVisited[row][col] = true;
        return 1 + dfs(row + 1, col) + dfs(row, col + 1);
    }

    //求数位和
    int bitSum(int n) {
        int sum = 0;
        while (n != 0) {
            sum += n % 10;
            n /= 10;
        }
        return sum;
    }
};
```



### [剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

给你二树的根节点 root 和一个整数目标和 targetSum ，找出所有 从根节点到叶子节点 路径总和等于给定目标和的路径。

叶子节点 是指没有子节点的节点。

 **题解**

DFS + 回溯。

满足叶子节点且路径和为零时，保存路径。

可用 target 减去路径值来代替每次计算路径和。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
    vector<vector<int>> ans;
    vector<int> path;
public:
    vector<vector<int>> pathSum(TreeNode *root, int target) {
        dfs(root, target);
        return ans;
    }

    void dfs(TreeNode *root, int target) {
        if (!root)
            return;
        //修改状态
        path.push_back(root->val);
        target -= root->val;
        //检查是否为叶子节点，并且 target 为零
        if (!root->left && !root->right && target == 0)
            ans.push_back(path);
        dfs(root->left, target);
        dfs(root->right, target);
        //回溯，恢复原状
        path.pop_back();
    }
};
```



### [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

 **题解**

DFS。

利用中序遍历生成有序序列。并维护 pre 值。

每次操作根节点时，将当前节点与 pre 值建立连接。

最后操作头尾节点。

**代码**

```c++
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* left;
    Node* right;

    Node() {}

    Node(int _val) {
        val = _val;
        left = NULL;
        right = NULL;
    }

    Node(int _val, Node* _left, Node* _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
    Node *pre, *head;
public:
    Node *treeToDoublyList(Node *root) {
        if (!root)
            return NULL;
        dfs(root);
        //最后处理头尾节点
        head->left = pre;
        pre->right = head;
        return head;
    }

    void dfs(Node *root) {
        if (!root)
            return;
        //递归操作左子树
        dfs(root->left);
        //当前节点操作
        if (!pre) {
            //如果 pre 为空，说明当前节点是头节点，赋值给 head
            head = root;
        } else {
            //如果不为空，将当前节点与 pre 建立连接
            pre->right = root;
            root->left = pre;
        }
        //更新 pre 的值为当前节点
        pre = root;
        //递归操作右子树
        dfs(root->right);
    }
};
```



### [剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)



给定一棵二叉搜索树，请找出其中第 k 大的节点的值。

**题解**

中序便利的逆序。

直接计数。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
    int ans, kNum;
public:
    int kthLargest(TreeNode *root, int k) {
        kNum = k;
        dfs(root);
        return ans;
    }

    void dfs(TreeNode *root) {
        if (!root)
            return;
        dfs(root->right);
        kNum--;
        if (kNum == 0) {
            ans = root->val;
            return;
        }
        dfs(root->left);
    }
};
```



### [剑指 Offer 55 - I. 二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

**题解**

经典问题，递归求解。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    int maxDepth(TreeNode *root) {
        if (!root)
            return 0;
        return 1 + max(maxDepth(root->left), maxDepth(root->right));
    }
};
```



### [剑指 Offer 55 - II. 平衡二叉树](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

**题解**



平衡二叉树

- 空树
- 节点的左右子树的深度相差不超过 1 且左右子树也是平衡二叉树

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    bool isBalanced(TreeNode *root) {
        if (!root)
            return true;
        //判断高度
        if (abs(maxDepth(root->left) - maxDepth(root->right)) > 1)
            return false;
        //判断左右子树
        return isBalanced(root->left) && isBalanced(root->right);
    }

    int maxDepth(TreeNode *root) {
        if (!root)
            return 0;
        return 1 + max(maxDepth(root->left), maxDepth(root->right));
    }
};
```



### [剑指 Offer 64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

求 `1+2+...+n` ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

**题解**

逻辑符短路。

**代码**

```c++
class Solution {
public:
    int sumNums(int n) {
        n > 1 && (n += sumNums(n - 1));
        return n;
    }
};
```



### [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

迭代。

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode *lowestCommonAncestor(TreeNode *root, TreeNode *p, TreeNode *q) {
        while (root) {
            if (root->val > p->val && root->val > q->val) {
                root = root->left;
            } else if (root->val < p->val && root->val < q->val) {
                root = root->right;
            } else
                break;
        }
        return root;
    }
};
```



### [剑指 Offer 68 - II. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

DFS

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode *lowestCommonAncestor(TreeNode *root, TreeNode *p, TreeNode *q) {
        if (root == nullptr || root == p || root == q) return root;
        TreeNode *left = lowestCommonAncestor(root->left, p, q);
        TreeNode *right = lowestCommonAncestor(root->right, p, q);
        if (left == nullptr) return right;
        if (right == nullptr) return left;
        return root;
    }
};
```



## 位运算

### [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

用每位去和 1 与，结果为 1 说明该位为 1。

**代码**

```c++
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int cnt = 0;
        for (int i = 0; i < 32; i++) {
            if (n & 1)
                cnt++;
            n = n >> 1;
        }
        return cnt;
    }
};
```



### [剑指 Offer 65. 不用加减乘除做加法](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

全加器的原理。

**代码**

```c++
class Solution {
public:
    int add(int a, int b) {
        while (b) {
            int carry = a & b; // 计算 进位
            a = a ^ b; // 计算 本位
            b = (unsigned)carry << 1;
        }
        return a;
    }
};
```



### [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

一个整型数组 `nums` 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

**题解**

先全部按位异或，结果就是两个数字异或的结果。

保存该结果中 1 出现的最低位，该位即是两个数字不同的最低位。

用该位将数组分组异或，得到的结果就是两个数。

**代码**

```c++
class Solution {
public:
    vector<int> singleNumbers(vector<int> &nums) {
        int z = 0;
        for (int i = 0; i < nums.size(); i++) {
            z ^= nums[i];
        }
        int m = 1;
        while (!(m & z))
            m <<= 1;
        int a = 0, b = 0;
        for (int i = 0; i < nums.size(); i++)
            if (m & nums[i])
                a ^= nums[i];
            else
                b ^= nums[i];
        return vector<int>{a, b};
    }
};
```



### [剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

在一个数组 `nums` 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

**题解**

按位加，模 3 后得到结果

**代码**

```c++
class Solution {
public:
    int singleNumber(vector<int> &nums) {
        vector<int> counts(32);
        for (int i = 0; i < nums.size(); i++) {
            for (int j = 0; j < 32; j++) {
                counts[j] += nums[i] & 1;
                nums[i] >>= 1;
            }
        }
        int ans = 0, MOD = 3;
        for (int i = 0; i < 32; i++) {
            ans <<= 1;
            ans |= counts[31 - i] % MOD;
        }
        return ans;
    }
};
```



## 栈与队列

### [剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

**题解**

维护两个栈，一个栈stack_in只负责输出，另一个栈stack_out只负责输入。

当stack_out为空时，将stack_in倒入stack_out中。

因为栈的先入后出特性，此过程完成了一次反转，变成了先入先出，符合队列的特性

**代码**

```c++
class CQueue {
    stack<int> stack_in, stack_out;
public:
    CQueue() {
        while (!stack_in.empty()) {
            stack_in.pop();
        }
        while (!stack_out.empty()) {
            stack_out.pop();
        }
    }

    void appendTail(int value) {
        stack_in.push(value);
    }

    int deleteHead() {
        //输出栈为空，将输入栈倒入输出栈中
        if (stack_out.empty()) {
            while (!stack_in.empty()) {
                stack_out.push(stack_in.top());
                stack_in.pop();
            }
        }
        //如果倾倒完，输出栈还为空，说明队列为空
        if (stack_out.empty()) {
            return -1;
        } else {
            int ret = stack_out.top();
            stack_out.pop();
            return ret;
        }
    }
};

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue* obj = new CQueue();
 * obj->appendTail(value);
 * int param_2 = obj->deleteHead();
 */
```



### [剑指 Offer 30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

**题解**



**解法一 保存最小元素信息**

由于栈的特性，每个元素退出、插入后、访问时，该栈的结构不会发生变化，所以我们可以在插入元素时存：**到该元素为止最小的元素**。

这样，每次访问一个元素时，当前栈中最小的元素值一定是该位置保存的最小元素值。

**代码**

```c++
class MinStack {
    struct MinNode {
        int min;
        int value;

        MinNode(int min, int value) : min(min), value(value) {}
    };

    typedef MinNode *PtrToNode;
    stack<PtrToNode> stack;

public:
    /** initialize your data structure here. */
    MinStack() {
        while (!stack.empty()) {
            stack.pop();
        }
    }

    void push(int x) {
        if (stack.empty()) {
            stack.push(new MinNode(x, x));
        } else {
            if (x < stack.top()->min) {
                stack.push(new MinNode(x, x));
            } else {
                stack.push(new MinNode(stack.top()->min, x));
            }
        }
    }

    void pop() {
        stack.pop();
    }

    int top() {
        return stack.top()->value;
    }

    int min() {
        return stack.top()->min;
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(x);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->min();
 */
```



**解法二 单调栈**

使用单调栈，我们维护栈中元素递减。

为了不丢失信息，我们在新元素不符合递减时不弹出栈中元素，而是在符合递减时才加入新元素。

**代码**

```c++
class MinStack {
    //递减栈
    stack<int> minStack;
    stack<int> stack;
public:
    /** initialize your data structure here. */
    MinStack() {
        while (!stack.empty())
            stack.pop();
        while (!minStack.empty()) {
            minStack.empty();
        }
    }

    void push(int x) {
        stack.push(x);
        if (minStack.empty() || x <= minStack.top())
            minStack.push(x);
    }

    void pop() {
        if (stack.empty())
            return;
        if (stack.top() == minStack.top())
            minStack.pop();
        stack.pop();
    }

    int top() {
        if (stack.empty())
            return 0;
        return stack.top();
    }

    int min() {
        if (stack.empty())
            return 0;
        return minStack.top();
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(x);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->min();
 */
```



## 字符串

### [剑指 Offer 05. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

 

**题解**

没啥好说的，直接遍历

**代码**

```c++
class Solution {
public:
    string replaceSpace(string s) {
        string ans;
        for (auto ch:s) {
            if (ch == ' ') {
                ans.push_back('%');
                ans.push_back('2');
                ans.push_back('0');
            } else {
                ans.push_back(ch);
            }
        }
        return ans;
    }
};
```



### [剑指 Offer 58 - II. 左旋转字符串](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。



**题解**

使用string类的substr()方法，直接拼接字符串。

**代码**

```c++
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        string ans;
        ans = s.substr(n);
        ans += s.substr(0, n);
        return ans;
    }
};
```



## 分治算法

### [剑指 Offer 07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

输入某二叉树的前序遍历和中序遍历的结果，请构建该二叉树并返回其根节点。

假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

**题解**

分治。

对于任意一颗树而言，前序遍历的形式总是

```
[ 根节点, [左子树的前序遍历结果], [右子树的前序遍历结果] ]
```


即根节点总是前序遍历中的第一个节点。而中序遍历的形式总是

```
[ [左子树的中序遍历结果], 根节点, [右子树的中序遍历结果] ]
```

- 使用 map 对应，便于定位该值在 inorder 中的位置。
- 左子树在前序中的根节点位于：`pre_root + 1`,左子树在中序中的边界：`[in_left, in_root - 1]`
- 右子树在前序中的根节点位于：根节点+左子树长度+1 = `pre_root + in_root - in_left + 1`，右子树在中序中的边界：`[in_root + 1, in_right]`

**代码**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
    map<int, int> map;
public:
    TreeNode *buildTree(vector<int> &preorder, vector<int> &inorder) {
        for (int i = 0; i < inorder.size(); i++) {
            map[inorder[i]] = i;
        }
        return build(preorder, inorder, 0, 0, inorder.size() - 1);
    }

    TreeNode *build(vector<int> &preorder, vector<int> &inorder, int pre_root, int in_left, int in_right) {
        if (in_left > in_right)
            return NULL;
        TreeNode *root = new TreeNode(preorder[pre_root]);
        // 根节点在中序序列中的位置，用于划分左右子树的边界
        int in_root = map[preorder[pre_root]];
        // 左子树在前序中的根节点位于：pre_root+1,左子树在中序中的边界：[in_left,in_root-1]
        root->left = build(preorder, inorder, pre_root + 1, in_left, in_root - 1);
        // 右子树在前序中的根节点位于：根节点+左子树长度+1 = pre_root+in_root-in_left+1
        // 右子树在中序中的边界：[in_root+1,in_right]
        root->right = build(preorder, inorder, pre_root + in_root - in_left + 1, in_root + 1, in_right);
        return root;
    }
};
```



### [剑指 Offer 16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

实现 pow(x, n) ，即计算 x 的 n 次幂函数（即，xn）。不得使用库函数，同时不需要考虑大数问题。

 

**题解**

快速幂。

通过对 1 左移去判断当前位需不需要乘进 ans 中。同时保持 x 的自增长。

**代码**

```c++
class Solution {
public:
    double myPow(double x, int n) {
        bool isNegative = false;
        long long N = n;
        if (N == 0)
            return 1;
        //对 n 取反
        if (N < 0)
            isNegative = true, N = -N;
        double ret = 1;
        for (int i = 0; i < 32; ++i) {
            if ((1 << i) & N)
                ret *= x;
            x = x * x;
        }
        //本质上的差别只是最后的返回值
        if (isNegative)
            return 1.0 / ret;
        else
            return ret;
    }
};
```



### [剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。

**题解**

分治，二叉搜索树的左子树应当都比根节点小，右子树应当都比根节点大。

所以我们可以把遇到的第一个比根节点大的数定义为右子树的左区间端点，如果其中再出现比根节点小的数，证明不是二叉搜索树。

同时需要左右子树都是二叉搜索数。

**代码**

```c++
class Solution {
public:
    bool verifyPostorder(vector<int> &postorder) {
        return recur(postorder, 0, postorder.size() - 1);
    }

    bool recur(vector<int> &postorder, int left, int right) {
        if (left >= right)
            return true;
        int p = left;
        while (postorder[p] < postorder[right])
            p++;
        int m = p;
        while (postorder[p] > postorder[right])
            p++;
        return p == right && recur(postorder, left, m - 1) && recur(postorder, m, right - 1);
    }
};
```



## 动态规划

### [剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)



写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项（即 F(N)）。斐波那契数列的定义如下：

```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```

斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**题解**

普通的递归会超时，因为有很多的重复计算，需要优化。

在这里我们使用动态规划。

维护一个滑动窗口，根据前两个值直接计算出第三个值。

**代码**

```c++
class Solution {
public:
    int fib(int n) {
        int MOD = 1e9 + 7;
        if (n < 2)
            return n;
        int first = 0, second = 0, third = 1;
        for (int i = 2; i <= n; ++i) {
            first = second;
            second = third;
            third = (first + second) % MOD;
        }
        return third;
    }
};
```



### [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)



一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**题解**

我们从后往前看，当青蛙要跳上 n 阶时，要么跳一步，要么跳两步。

所以 n 阶的跳法就是 n-1 阶和 n-2阶的和。

```
dp[i]=dp[i-1] + dp[i-2]
```

本体与斐波那契数列类似，都可以用动态规划。

注意边界条件

> dp[0]=1
>
> dp[1]=1

**代码**

```c++
class Solution {
public:
    int numWays(int n) {
        int MOD = 1e9 + 7;
        if (n < 2)
            return 1;
        int first = 0, second = 1, third = 1;
        for (int i = 2; i <= n; ++i) {
            first = second;
            second = third;
            third = (first + second) % MOD;
        }
        return third;
    }
};
```



### [剑指 Offer 63. 股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)



假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

**题解**

直接保存历史最低价格，与每天的价格比较，记录最大利润。

**代码**

```c++
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int min = INT_MAX, ans = 0;
        for (int i = 0; i < prices.size(); ++i) {
            if (prices[i] < min)
                min = prices[i];
            if (prices[i] - min > ans)
                ans = prices[i] - min;
        }
        return ans;
    }
};
```





### [剑指 Offer 42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)



输入一个整型数组，数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。

要求时间复杂度为O(n)。

**题解**

动态规划。

dp数组存到每个位置为止的最大子列和。

```c++
dp[i] = max(dp[i - 1] + nums[i], nums[i]);
```

将前一位的最大子列和与本位相加

- 如果小于本位，则说明本位可另起最大子列。
- 如果大于本位，则本位储存该结果，表示到该位为止的最大子列和



**代码**

```c++
class Solution {
public:
    int maxSubArray(vector<int> &nums) {
        vector<int> dp(nums.size());
        int ans = nums[0];
        for (int i = 0; i < nums.size(); ++i) {
            //边界判断
            if (i == 0) {
                dp[0] = nums[0];
                continue;
            }
            //递推关系，比较本位和本位加上前一位的最大子列和，得出本位的最大子列和
            dp[i] = max(dp[i - 1] + nums[i], nums[i]);
            //储存结果
            if (dp[i] > ans)
                ans = dp[i];
        }
        return ans;
    }
};
```



### [剑指 Offer 47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)



在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

**题解**

动态规划。

二维dp，dp矩阵的每个值储存到该位置为止，拿到的最大价值。

因为每个位置可从上边或者左边拿，所以比较两个位置的最大值，再加上本位的价值。

```c++
dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]) + grid[i - 1][j - 1];
```

可以用新数组，最左列和最上列初始化为0，可省略边界判断。

**代码**

```c++
class Solution {
public:
    int maxValue(vector<vector<int>> &grid) {
        vector<vector<int>> dp(grid.size() + 1);
        //初始化，避免边界判断
        for (int i = 0; i < dp.size(); ++i) {
            dp[i] = vector<int>(grid[0].size() + 1);
        }
        int ans = 0;
        for (int i = 1; i < dp.size(); ++i) {
            for (int j = 1; j < dp[i].size(); ++j) {
                //递推关系，从左和上两个方向选出最大值，加上本位价值，得出到本位的最大价值
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]) + grid[i - 1][j - 1];
                //保存结果
                ans = max(dp[i][j], ans);
            }
        }
        return ans;
    }
};
```





### [剑指 Offer 46. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)



给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。

**题解**

类似于跳台阶，区别在于，需要判断能不能跳两个台阶。

dp 数组储存到当前位置为止的跳法数量。

```c++
class Solution {
public:
    int translateNum(int num) {
        string str = to_string(num);
        if (str.size() == 1)
            return 1;
        vector<int> dp(str.length());
        dp[0] = 1;
        if (stoi(str.substr(0, 2)) < 26)
            dp[1] = 2;
        else
            dp[1] = 1;
        for (int i = 2; i < str.length(); ++i) {
            int num_two = stoi(str.substr(i - 1, 2));
            if (num_two > 9 && num_two < 26)
                dp[i] = dp[i - 1] + dp[i - 2];
            else
                dp[i] = dp[i - 1];
        }
        return dp[str.length() - 1];
    }
};
```



### [剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)



请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。

**题解**

动态规划。

使用 dp 数组存到当前位置为止，最长不重复子串。用 map 存每个字符在当前子串里的位置。

当遇到重复字符，我们用当前位置减去字符在上一个子串里的位置得到裁剪出的新串长度。

但如果裁剪出的新串长度大于累加得到的新串长度，说明该字符已经在过去被裁剪过，现在只要累加就好。

**代码**

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s.empty())
            return 0;
        map<char, int> map;
        vector<int> dp(s.size());
        dp[0] = 1;
        map[s[0]] = 0;
        int ans = 1;
        for (int i = 1; i < s.size(); ++i) {
            if (map.count(s[i]) > 0 && i - map[s[i]] < dp[i - 1]+1) {
                //裁剪出的新串长度小于累加得到的新串长度，说明该字符在过去被裁剪过
                dp[i] = i - map[s[i]];
            } else
                dp[i] = dp[i - 1] + 1;
            ans = max(dp[i], ans);
            map[s[i]] = i;
        }
        return ans;
    }
};
```



## 查找算法

### [剑指 Offer 03. 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

**题解**

**解法一**

遍历，保存到set里

**代码**

```c++
class Solution {
    set<int> set;
public:
    int findRepeatNumber(vector<int> &nums) {
        for (auto item:nums) {
            if (set.count(item) > 0)
                return item;
            set.insert(item);
        }
        //题目保证有重复，这里返回什么都可以
        return -1;
    }
};
```

**解法二**

原地交换。

遍历时，将元素与元素值对应的索引地址交换，形成 `nums[i] == nums[nums[i]]` 。

这样，下次遇到相同的 `nums[i]` 想交换时，会发现目标位置已经有重复值，最后返回这个重复值

**注意**

如果进行了一次交换，`i` 不要增加，因为不能保证交换过来的元素（也就是 `i` 位置当前的元素）处于正确的位置，所以我们需要再对 `i` 位置判断一次。

**代码**

```c++
class Solution {
public:
    int findRepeatNumber(vector<int> &nums) {
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] != i) {
                if (nums[i] == nums[nums[i]])
                    return nums[i];
                swap(nums[i], nums[nums[i]]);
                //i不增加，对当前位置再判断一次
                i--;
            }
        }
        return -1;
    }
};
```



### [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/)

统计一个数字在排序数组中出现的次数。

**题解**

两次二分，分别找 `target` 的上界和下界，返回差值。

注意二分的边界和落点判断。

**代码**

```c++
class Solution {
public:
    int search(vector<int> &nums, int target) {
        int left = 0, right = nums.size() - 1, mid;
        while (left <= right) {
            mid = (left + right) / 2;
            if (nums[mid] < target)
                left = mid + 1;
            else
                right = mid - 1;
        }
        int lBound = left;
        left = 0, right = nums.size() - 1;
        while (left <= right) {
            mid = (left + right) / 2;
            if (nums[mid] <= target)
                left = mid + 1;
            else
                right = mid - 1;
        }
        int rBound = left;
        return rBound - lBound;
    }
};
```





### [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/)

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

**题解**

排序数组，优先使用二分法。

思考缺失的位置有什么特殊点（可比较点）。

缺失的位置左边都应是 `nums[i]=i` ，所以我们直接寻找该序列的最右端，最右端右侧的位置就是缺失的位置。

注意二分的边界和落点判断。

**代码**

```c++
class Solution {
public:
    int missingNumber(vector<int> &nums) {
        int left = 0, right = nums.size() - 1, mid;
        while (left <= right) {
            mid = (left + right) / 2;
            if (nums[mid] == mid)
                left = mid + 1;
            else
                right = mid - 1;
        }
        return left;
    }
};
```



### [剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

 

**题解**

参考：[面试题04. 二维数组中的查找（标志数，清晰图解） - 二维数组中的查找 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/solution/mian-shi-ti-04-er-wei-shu-zu-zhong-de-cha-zhao-zuo/)

我们发现此排序矩阵类似二叉搜索树，对于每个元素，其“左分支”元素更小，“右分支”元素更大。

![Picture1.png](https://cdn.jsdelivr.net/gh/TBDGF/TBDGF.github.io@master/img/970c/6584ea93812d27112043d203ea90e4b0950117d45e0452d0c630fcb247fbc4af-Picture1.png)

所以我们可以从左下角或右上角开始，按照类似二叉搜索树方式遍历。

**代码**

```c++
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>> &matrix, int target) {
        if (matrix.size() == 0)
            return false;
        //这里我们选的从右上角开始遍历
        int col = matrix[0].size() - 1, row = 0;
        int curr;
        //设置边界条件，防止越界
        while (col >= 0 && row <= matrix.size() - 1) {
            curr = matrix[row][col];
            if (curr == target)
                return true;
            if (target > curr)
                row++;
            else
                col--;
        }
        return false;
    }
};
```



### [剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。

给你一个可能存在 重复 元素值的数组 numbers ，它原来是一个升序排列的数组，并按上述情形进行了一次旋转。请返回旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一次旋转，该数组的最小值为1。  

**题解**

本题为排序数组，所以用二分解。

二分的中心思想是不断地缩小边界，我们需要考虑各个情况如何缩小边界，最后到达左侧排序序列的右边界。

**首先判断mid落在哪里**

我们默认在到达边界之前，left都应在左序列，而right在右序列

- 如果mid落在左排序序列

此时mid大于等于left，同时也大于等于right。

- 如果mid落在右排序序列

此时mid小于等于left，同时也小于等于right。

由上可知，只需要单独关注left或者right就行，并且需要区分出相等的情况。在本题解，我们关注right。

- 当mid等于right时

我们只需要让right减小，以缩小边界。

**代码**

```c++
class Solution {
public:
    int minArray(vector<int> &numbers) {
        if (numbers.size() == 1)
            return numbers[0];
        int left = 0, right = numbers.size() - 1, mid;
        while (left < right) {
            mid = left + (right - left) / 2;
            if (numbers[mid] < numbers[right])
                right = mid;
            else if (numbers[mid] > numbers[right])
                left = mid + 1;
            else
                right--;
        }
        return numbers[left];
    }
};
```



### [剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。

**题解**

首先用map存储字符出现的频次，如果多次出现则置为-1。

其次我们需要第一个只出现一次的字符，很适合用队列（FIFO）

在每个字符第一次出现的时候，将其加入队列。

遍历完频次后，从队列头开始判断，如果value为-1说明多次出现，出队列。

第一个value为1的元素即是第一个出现一次的元素。

**代码**

```c++
class Solution {
    map<char, int> map;
    queue<char> queue;
public:
    char firstUniqChar(string s) {
        if (s.size() == 0)
            return ' ';
        for (auto ch:s) {
            if (map.count(ch) == 0) {
                map[ch] = 1;
                queue.push(ch);
            } else {
                map[ch] = -1;
            }
        }
        while (!queue.empty() && map[queue.front()] == -1)
            queue.pop();
        if (queue.empty())
            return ' ';
        else
            return queue.front();
    }
};
```

