## priority_queue 优先队列（堆）

默认为大根堆

```c++
priority_queue<int> heap;
```

可指定排序函数

```c++
//大根堆
priority_queue<int, vector<int>, less<int>> maxHeap;

//小根堆
priority_queue<int, vector<int>, greater<int>> minHeap;
```

操作

- top 访问队头元素
- empty 队列是否为空
- size 返回队列内元素个数
- push 插入元素到队尾 (并排序)
- emplace 原地构造一个元素并插入队列
- pop 弹出队头元素
- swap 交换内容



## 异或

异或运算有以下三个性质。

任何数和 0 做异或运算，结果仍然是原来的数。

任何数和其自身做异或运算，结果是 0。

异或运算满足交换律和结合律。



## 随机数

随机整数

```c++
rand()
```

随机 0 ~ n 的整数

```c++
rand() % n
```

