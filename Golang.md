## 参数传递

默认采用值传递，且Go中函数传参仅有值传递一种方式。

slice、map、channel都是引用类型。

slice能够通过函数传参后，修改对应的数组值，是因为slice内部保存了引用数组的指针，并不是因为引用传递。



## Go协程

**Go协程与Java线程的区别**

- 协程比线程更细粒度，协程运行在线程之上，多个协程可以由一个或多个线程管理。
- 协程上下文切换较快，没有线程之间切换的开销。
- 协程的调度不需要多线程的锁机制，因为只有一个线程，不存在同时写变量冲突，执行效率比多线程高很多。

**Golang 的协程间通讯方式有哪些**

- 共享内存和channel通信。
- 「Don’t communicate by sharing memory, share memory by communicating」所以更提倡使用 channel 进行通信。



## GMP模型

**G**

G是`Goroutine`的缩写。它包含运行栈+寄存器数值（PC、BP、SP）。

协程的切换，仅仅需要改变寄存器的数值，cpu便会从需要切换的协程指定位置继续运行。

```go
type g struct {

    m            *m      // current m; offset known to arm liblink
    sched        gobuf
    ...
    param        unsafe.Pointer // passed parameter on wakeup
    goid         int64
    ...
    vdsoSP uintptr // SP for traceback while in VDSO call (0 if not in call)
    vdsoPC uintptr // PC for traceback while in VDSO call
}
```

**M**

M（线程，machine）是一个线程，每个M都有一个线程的栈。如果没有给线程的栈分配内存，操作系统会给线程的栈分配默认的内存。当线程的栈制定，M.stack->G.stack, M的PC寄存器会执行G提供的函数。

```go
type m struct {    
    /*
    g0的线程栈与M相关
    */
    g0       *g
    Curg *g //M 现在绑定的G

    // SP, PC registers for on-site protection and on-site recovery
    vdsoSP uintptr
    vdsoPC uintptr
    ...
}
```

**P**

P(处理器，Processor)是一个抽象的概念，不是物理上的CPU。当一个P有任务，需要创建或者唤醒一个系统线程去处理它队列中的任务。

P决定同时执行的任务的数量，`GOMAXPROCS`限制系统线程执行用户层面的任务的数量。

**调度过程**

首先创建一个G对象，然后G被保存在P的本地队列或者全局队列（global queue）。这时P会唤醒一个M。P按照它的执行顺序继续执行任务。M寻找一个空闲的P，如果找得到，将G移动到它自己。然后M执行一个调度循环：调用G对象->执行->清理线程->继续寻找Goroutine。

在M的执行过程中，上下文切换随时发生。当切换发生，任务的执行现场需要被保护，这样在下一次调度执行可以进行现场恢复。M的栈保存在G对象，只有现场恢复需要的寄存器(SP,PC等)，需要被保存到G对象。

如果G对象还没有被执行，M可以将G重新放到P的调度队列，等待下一次的调度执行。当调度执行时，M可以通过G的vdsoSP, vdsoPC 寄存器进行现场恢复。

P队列 P有2种类型的队列：

1. 本地队列：本地的队列是无锁的，没有数据竞争问题，处理速度比较高。
2. 全局队列：是用来平衡不同的P的任务数量，所有的M共享P的全局队列。

线程清理 G的调度是为了实现P/M的绑定，所以线程清理就是释放P上的G，让其他的G能够被调度。

1. 主动释放(active release)：典型的例子是，执行G任务时，发生了系统调用(system call)，这时M会处于阻塞（Block）状态。调度器会设置一个超时时间，来释放P。
2. 被动释放(passive release)：如果系统调用发生，监控程序需要扫描处于阻塞状态的P/M。 这时，超时之后，P资源会回收，程序被安排给队列中的其他G任务。

**调度示意图**

![img](https://pic4.zhimg.com/80/v2-d1d7e384605ff9b2d0a83b9bcd4a8247_720w.jpg)

### **调度策略**

**复用线程**

- wrok stealing机制：当本线程无可运行的G时，尝试从其他线程绑定的P偷取G，而不是销毁线程。
- hand off机制：当本线程因为G进行系统调用阻塞时，线程释放绑定的P，把P转移给其他空闲的线程执行。

**抢占式调度(Preemptive scheduling)**

限制协程执行时长，不会出现饿死现象

### go func(){}之后

<img src="https://pic3.zhimg.com/80/v2-a9082b3ab006addff05b6c759d6d67f6_720w.jpg" alt="img" style="zoom:150%;" />

1. `go func(){}`创建一个新的`goroutine`
2. G保存在P的本地队列，如果本地队列满了，保存在全局队列
3. G在M上运行，每个M绑定一个P。如果P的本地队列没有G，M会从其他P的本地队列，或者G的全局队列，窃取G
4. 当M阻塞时，会将M从P解除。把G运行在其他空闲的M或者创建新的M。
5. 当M恢复时，会尝试获得一个空闲的P。如果没有P空闲，M会休眠，G会放到全局队列。
