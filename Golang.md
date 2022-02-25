## 参数传递

默认采用值传递，且Go中函数传参仅有值传递一种方式。

slice、map、channel都是引用类型。

slice能够通过函数传参后，修改对应的数组值，是因为slice内部保存了引用数组的指针，并不是因为引用传递。



## Go协程与Java线程的区别

- 协程比线程更细粒度，协程运行在线程之上，多个协程可以由一个或多个线程管理。
- 协程上下文切换较快，没有线程之间切换的开销。
- 协程的调度不需要多线程的锁机制，因为只有一个线程，不存在同时写变量冲突，执行效率比多线程高很多。



## Golang 的协程间通讯方式有哪些

- 共享内存和channel通信。
- 「Don’t communicate by sharing memory, share memory by communicating」所以更提倡使用 channel 进行通信。

