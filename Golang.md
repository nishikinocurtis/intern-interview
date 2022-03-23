## Go优点

1.简单高效：几乎所有主流的编程语言如Java、C++、PHP、Python、JavaScript等等都是可以用于服务端开发的，Go语言作为后起之秀，在语言层面具有语法简洁、执行效率高（底层语言）的特点；相比之下，Java和Python、PHP都显得低效，C++则太过麻烦，而Go则可以做到简单与高效兼顾；

2.高并发：Go语言是主打并发、为并发而生的，其出发点即是瞄准大数据+云计算时代背景下的高并发、分布式应用场景；

3.跨平台：可以在不同平台直接编译生成可执行程序，基础内存占用很少，小应用占用几M大型应用占用个几十M就能很好运行，这使得golang可以在树莓派之类的小设备上很好的运行，这一点表现比java要好的多；



## 参数传递

默认采用值传递，且Go中函数传参仅有值传递一种方式。

slice、map、channel都是引用类型。

slice能够通过函数传参后，修改对应的数组值，是因为slice内部保存了引用数组的指针，并不是因为引用传递。



## Go协程

**Go协程与Java线程的区别**

- Go的协程是一个轻量级的线程，多个协程可能运行在一个或者多个线程上；
- Go的协程是非抢占式多任务处理，需要由协程主动交出控制权，这与其他抢占型的线程并发方式不同，这种非抢占式设计减少了许多cpu调度切换的开销；
- Go的协程是一个虚拟机层面的多任务处理，它基于Go的并发调度器，其调度是go运行时自动管理的，对用户不可见，用户只需管理业务层面的并发即可。
- Go中所有的调用栈都是一个协程，main()函数就是主协程，如无开辟其他协程所有任务都在主协程运行。

**Golang 的协程间通讯方式有哪些**

- 共享内存和channel通信。
- 「Don’t communicate by sharing memory, share memory by communicating」所以更提倡使用 channel 进行通信。



## channel

channel 是一个引用类型，所以在它被初始化之前，它的值是 nil，channel 使用 make 函数进行初始化。可以向它传递一个 int 值，代表 channel 缓冲区的大小（容量），构造出来的是一个缓冲型的 channel；不传或传 0 的，构造的就是一个非缓冲型的 channel。



## 同步机制

**等待组 sync.WaitGroup**

Go同步包sync提供等待组，以解决主协程等待子协程完成任务的问题。

```go
func waitGroup() {
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(i int) {
			t := rand.Intn(3)
			time.Sleep(time.Duration(t) * time.Second)
			fmt.Printf("The %d thing is Done.\n", i)
			wg.Done()
		}(i)
	}
	wg.Wait()
	fmt.Println("Finished!")
}
```

**同步锁/互斥锁 sync.Mutex**

所谓互斥锁，sync.Mutex ,保证被锁定资源不被其他协程占用，即被加锁的对象在同一时间只允许一个协程读或写。

```go
func BaseSync02() {
	//申请一个锁
	mutex := sync.Mutex{}
	num := 200

	//开100个协程
	for i := 1; i <= 100; i++ {
		go func(n int) {
			//每个协程分100次对num操作
			for j := 1; j <= 100; j++ {
				mutex.Lock()
				num += 1
				mutex.Unlock()
			}
		}(i)
	}

	time.Sleep(time.Second * 3)
	fmt.Println("num：", num)
}
```

Mutex实现中有两种模式，**1：正常模式，2：饥饿模式**，前者指的是当一个协程获取到锁时，后面的协程会排队(FIFO),释放锁时会唤醒最早排队的协程，这个协程会和正在CPU上运行的协程竞争锁，但是大概率会失败，为什么呢？因为你是刚被唤醒的，还没有获得CPU的使用权，而CPU正在执行的协程肯定比你有优势，如果这个被唤醒的协程竞争失败，并且超过了1ms，那么就会退回到后者(饥饿模式)，这种模式下，该协程在下次获取锁时直接得到,不存在竞争关系，本质是为了防止协程等待锁的时间太长。

**读写锁 snyc.RWMutex**

所谓读协程snyc.RWMutex，实现业务中对资源一写多读的情况 ，如对数据库读写，为保证数据原子性，同一时间只允许一个协程写资源，禁止其他写入或读取，或同一时间允许多个协程读取资源，但禁止任何协程写入。

**只执行一次 sync.Once**

多协程调用中对一个任务只允许执行一次

**条件变量 sync.Cond**



**原子操作 sync.atomic**

物理级别实现资源读写的原子性，从根本上杜绝并发不安全的问题，但其主要缺陷是只支持对基本数据类型的操作，对其他类型则无能为力。



## GMP模型

**G**

表示Goroutine，每个Goroutine对应一个G结构体，G存储Goroutine的运行堆栈、状态以及任务函数，可重用。

G运行队列是一个栈结构，分全局队列和P绑定的局部队列，每个G不能独立运行，它需要绑定到P才能被调度执行。



**M**

Machine，系统物理线程，代表着真正执行计算的资源，在绑定有效的P后，进入schedule循环；

而schedule循环的机制大致是从Global队列、P的Local队列以及wait队列中获取G，切换到G的执行栈上并执行G的函数，调用goexit做清理工作并回到M，如此反复。M并不保留G状态，这是G可以跨M调度的基础，M的数量是不定的，由Go Runtime调整，为了防止创建过多OS线程导致系统调度不过来，目前默认最大限制为10000个。



**P**

Processor，表示逻辑处理器， 对G来说，P相当于CPU核，G只有绑定到P(在P的local runq中)才能被调度。

对M来说，P提供了相关的执行环境(Context)，如内存分配状态(mcache)，任务队列(G)等，P的数量决定了系统内最大可并行的G的数量（前提：物理CPU核数 >= P的数量），P的数量由用户设置的GOMAXPROCS决定，但是不论GOMAXPROCS设置为多大，P的数量最大为256。



**调度过程**

- go关键字创建goroutine(G)，优先加入某个P维护的局部队列（当局部队列已满时才加入全局队列）；
- P需要持有或者绑定一个M，而M会启动一个系统线程，不断的从P的本地队列取出G并执行；
- M执行完P维护的局部队列后，它会尝试从全局队列寻找G，如果全局队列为空，则从其他的P维护的队列里窃取一般的G到自己的队列；
- 重复以上直到所有的G执行完毕。



**系统调度引起阻塞**

如系统GC，M会解绑P，出让控制权给其他M，让该P维护的G运行队列不至于阻塞。

**用户态的阻塞**

当goroutine因为管道操作或者系统IO、网络IO而阻塞时，对应的G会被放置到某个等待队列，该G的状态由运行时变为等待状态，而M会跳过该G尝试获取并执行下一个G，如果此时没有可运行的G供M运行，那么M将解绑P，并进入休眠状态；

当阻塞的G被另一端的G2唤醒时，如管道通知，G又被标记为可运行状态，尝试加入G2所在P局部队列的队头，然后再是G全局队列。



**调度示意图**

![img](https://pic4.zhimg.com/80/v2-d1d7e384605ff9b2d0a83b9bcd4a8247_720w.jpg)

### **调度策略**

**复用线程**

- work stealing机制：当本线程无可运行的G时，尝试从其他线程绑定的P偷取G，而不是销毁线程。
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



## Go 内存管理

Golang 的内存管理本质上就是一个内存池，只不过内部做了很多的优化。比如自动伸缩内存池大小，合理的切割内存块等等。

### 内存池 mheap

Golang 的程序在启动之初，会一次性从操作系统那里申请一大块内存作为内存池。这块内存空间会放在一个叫 `mheap` 的 `struct` 中管理，mheap 负责将这一整块内存切割成不同的区域，并将其中一部分的内存切割成合适的大小，分配给用户使用。

我们需要先知道几个重要的概念：

- **`page`**: 内存页，一块 `8K` 大小的内存空间。Go 与操作系统之间的内存申请和释放，都是以 `page` 为单位的。
- **`span`**: 内存块，**一个或多个连续的** `page` 组成一个 `span`。如果把 `page` 比喻成工人，`span` 可看成是小队，工人被分成若干个队伍，不同的队伍干不同的活。
- **`sizeclass`**: 空间规格，每个 `span` 都带有一个 `sizeclass`，标记着该 `span` 中的 `page` 应该如何使用。使用上面的比喻，就是 `sizeclass` 标志着 `span` 是一个什么样的队伍。
- **`object`**: 对象，用来存储一个变量数据内存空间，一个 `span` 在初始化时，会被切割成一堆**等大**的 `object`。假设 `object` 的大小是 `16B`，`span` 大小是 `8K`，那么就会把 `span` 中的 `page` 就会被初始化 `8K / 16B = 512` 个 `object`。所谓内存分配，就是分配一个 `object` 出去。

示意图：

![img](https:////upload-images.jianshu.io/upload_images/11662994-8361f3be115cf456.png?imageMogr2/auto-orient/strip|imageView2/2/w/404/format/webp)

上图中，不同颜色代表不同的 `span`，不同 `span` 的 `sizeclass` 不同，表示里面的 `page` 将会按照不同的规格切割成一个个等大的 `object` 用作分配。

使用 Go1.11.5 版本测试了下初始堆内存应该是 `64M` 左右，低版本会少点。

测试代码：



```go
package main
import "runtime"
var stat runtime.MemStats
func main() {
    runtime.ReadMemStats(&stat)
    println(stat.HeapSys)
}
```

内部的整体内存布局如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/11662994-356f568da2987e54.png?imageMogr2/auto-orient/strip|imageView2/2/w/817/format/webp)

- `mheap.spans`：用来存储 `page` 和 `span` 信息，比如一个 span 的起始地址是多少，有几个 page，已使用了多大等等。
- `mheap.bitmap` 存储着各个 `span` 中对象的标记信息，比如对象是否可回收等等。
- `mheap.arena_start`: 将要分配给应用程序使用的空间。

再说明下，图中的空间大小，是 Go 向操作系统申请的虚拟内存地址空间，操作系统会将该段地址空间预留出来不做它用；而不是真的创建出这么大的虚拟内存，在页表中创建出这么大的映射关系。

### mcentral

**用途相同**的 `span` 会以链表的形式组织在一起。 这里的用途用 `sizeclass` 来表示，就是指该 `span` 用来存储哪种大小的对象。比如当分配一块大小为 `n` 的内存时，系统计算 `n` 应该使用哪种 `sizeclass`，然后根据 `sizeclass` 的值去找到一个可用的 `span` 来用作分配。其中 `sizeclass` 一共有 67 种（Go1.5 版本，后续版本可能会不会改变不好说），如图所示：

![img](https:////upload-images.jianshu.io/upload_images/11662994-730fc9b0a604aea1.png?imageMogr2/auto-orient/strip|imageView2/2/w/551/format/webp)

找到合适的 `span` 后，会从中取一个 `object` 返回给上层使用。这些 `span` 被放在一个叫做 mcentral 的结构中管理。

mheap 将从 OS 那里申请过来的内存初始化成一个大 `span`(sizeclass=0)。然后根据需要从这个大 `span` 中切出小 `span`，放在 mcentral 中来管理。大 `span` 由 `mheap.freelarge` 和 `mheap.busylarge` 等管理。如果 mcentral 中的 `span` 不够用了，会从 `mheap.freelarge` 上再切一块，如果 `mheap.freelarge` 空间不够，会再次从 OS 那里申请内存重复上述步骤。下面是 mheap 和 mcentral 的数据结构：



```go
type mheap struct {
    // other fields
    lock      mutex
    free      [_MaxMHeapList]mspan // free lists of given length， 1M 以下
    freelarge mspan                // free lists length >= _MaxMHeapList, >= 1M
    busy      [_MaxMHeapList]mspan // busy lists of large objects of given length
    busylarge mspan                // busy lists of large objects length >= _MaxMHeapList

    central [_NumSizeClasses]struct { // _NumSizeClasses = 67
        mcentral mcentral
        // other fields
    }
    // other fields
}

// Central list of free objects of a given size.
type mcentral struct {
    lock      mutex // 分配时需要加锁
    sizeclass int32 // 哪种 sizeclass
    nonempty  mspan // 还有可用的空间的 span 链表
    empty     mspan // 没有可用的空间的 span 列表
}
```

这种方式可以避免出现外部碎片*（文章最后面有外部碎片的介绍）*，因为同一个 span 是按照固定大小分配和回收的，不会出现不可利用的一小块内存把内存分割掉。这个设计方式与现代操作系统中的伙伴系统有点类似。

### mcache

如果你阅读的比较仔细，会发现上面的 mcentral 结构中有一个 lock 字段；因为并发情况下，很有可能多个线程同时从 mcentral 那里申请内存的，必须要用锁来避免冲突。

但锁是低效的，在高并发的服务中，它会使内存申请成为整个系统的瓶颈；所以在 mcentral 的前面又增加了一层 mcache。

每一个 mcache 和每一个处理器(P) 是一一对应的，也就是说每一个 P 都有一个 mcache 成员。 Goroutine 申请内存时，首先从其所在的 P 的 mcache 中分配，如果 mcache 没有可用 `span`，再从 mcentral 中获取，并填充到 mcache 中。

从 mcache 上分配内存空间是不需要加锁的，因为在同一时间里，一个 P 只有一个线程在其上面运行，不可能出现竞争。没有了锁的限制，大大加速了内存分配。

所以整体的内存分配模型大致如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/11662994-e6d7200368ec06b6.png?imageMogr2/auto-orient/strip|imageView2/2/w/696/format/webp)

### 其他优化

#### zero size

有一些对象所需的内存大小是0，比如 `[0]int`, `struct{}`，这种类型的数据根本就不需要内存，所以没必要走上面那么复杂的逻辑。

系统会直接返回一个固定的内存地址。源码如下：



```go
func mallocgc(size uintptr, typ *_type, flags uint32) unsafe.Pointer {
    // 申请的 0 大小空间的内存
    if size == 0 {
        return unsafe.Pointer(&zerobase)
    }
    //.....
}
```

测试代码：



```go
package main

import (
    "fmt"
)

func main() {
    var (
        a struct{}
        b [0]int
        c [100]struct{}
        d = make([]struct{}, 1024)
    )
    fmt.Printf("%p\n", &a)
    fmt.Printf("%p\n", &b)
    fmt.Printf("%p\n", &c)
    fmt.Printf("%p\n", &(d[0]))
    fmt.Printf("%p\n", &(d[1]))
    fmt.Printf("%p\n", &(d[1000]))
}
// 运行结果，6 个变量的内存地址是相同的:
0x1180f88
0x1180f88
0x1180f88
0x1180f88
0x1180f88
0x1180f88
```

#### Tiny对象

上面提到的 `sizeclass=1` 的 span，用来给 `<= 8B` 的对象使用，所以像 `int32`, `byte`, `bool` 以及小字符串等常用的微小对象，都会使用 `sizeclass=1` 的 span，但分配给他们 `8B` 的空间，大部分是用不上的。并且这些类型使用频率非常高，就会导致出现大量的内部碎片。

所以 Go 尽量不使用 `sizeclass=1` 的 span， 而是将 `< 16B` 的对象为统一视为 tiny 对象(tinysize)。分配时，从 `sizeclass=2` 的 span 中获取一个 `16B` 的 object 用以分配。如果存储的对象小于 `16B`，这个空间会被暂时保存起来 (`mcache.tiny` 字段)，下次分配时会复用这个空间，直到这个 object 用完为止。

如图所示：

![img](https:////upload-images.jianshu.io/upload_images/11662994-d026190322b2c139.png?imageMogr2/auto-orient/strip|imageView2/2/w/479/format/webp)

以上图为例，这样的方式空间利用率是 `(1+2+8) / 16 * 100% = 68.75%`，而如果按照原始的管理方式，利用率是 `(1+2+8) / (8 * 3) = 45.83%`。
 源码中注释描述，说是对 tiny 对象的特殊处理，平均会节省 `20%` 左右的内存。

如果要存储的数据里有指针，即使 `<= 8B` 也不会作为 tiny 对象对待，而是正常使用 `sizeclass=1` 的 `span`。

#### 大对象

如上面所述，最大的 sizeclass 最大只能存放 `32K` 的对象。如果一次性申请超过 `32K` 的内存，系统会直接绕过 mcache 和 mcentral，直接从 mheap 上获取，mheap 中有一个 `freelarge` 字段管理着超大 span。

### 总结

内存的释放过程，没什么特别之处。就是分配的返过程，当 mcache 中存在较多空闲 span 时，会归还给 mcentral；而 mcentral 中存在较多空闲 span 时，会归还给 mheap；mheap 再归还给操作系统。这里就不详细介绍了。

总结一下，这种设计之所以快，主要有以下几个优势：

1. 内存分配大多时候都是在用户态完成的，不需要频繁进入内核态。
2. 每个 P 都有独立的 span cache，多个 CPU 不会并发读写同一块内存，进而减少 CPU L1 cache 的 cacheline 出现 dirty 情况，增大 cpu cache 命中率。
3. 内存碎片的问题，Go 是自己在用户态管理的，在 OS 层面看是没有碎片的，使得操作系统层面对碎片的管理压力也会降低。
4. mcache 的存在使得内存分配不需要加锁。

当然这不是没有代价的，Go 需要预申请大块内存，这必然会出现一定的浪费，不过好在现在内存比较廉价，不用太在意。

总体上来看，Go 内存管理也是一个金字塔结构：

![img](https:////upload-images.jianshu.io/upload_images/11662994-6d4f174886374a83.png?imageMogr2/auto-orient/strip|imageView2/2/w/484/format/webp)

这种设计比较通用，比如现在常见的 web 服务设计，为提升系统性能，一般都会设计成  `客户端 cache -> 服务端 cache -> 服务端 db` 这几层（当然也可能会加入更多层），也是金字塔结构。

**将有限的计算资源布局成金字塔结构，再将数据从热到冷分为几个层级，放置在金字塔结构上。调度器不断做调整，将热数据放在金字塔顶层，冷数据放在金字塔底层。**

这种设计利用了计算的局部性特征，认为冷热数据的交替是缓慢的。所以最怕的就是，数据访问出现冷热骤变。在操作系统上我们称这种现象为*内存颠簸*，系统架构上通常被说成是*缓存穿透*。其实都是一个意思，就是过度的使用了金字塔低端的资源。

这套内部机制，使得开发高性能服务容易很多，通俗来讲就是坑少了。一般情况下你随便写写性能都不会太差。我遇到过的导致内存分配出现压力的主要有 2 中情况：

1. 频繁申请大对象，常见于文本处理，比如写一个海量日志分析的服务，很多日志内容都很长。这种情况建议自己维护一个对象([]byte)池，避免每次都要去 mheap 上分配。
2. 滥用指针，指针的存在不仅容易造成内存浪费，对 GC 也会造成额外的压力，所以尽量不要使用指针。



## Go GC

[图解Golang的GC算法 - Go语言中文网 - Golang中文社区 (studygolang.com)](https://studygolang.com/articles/18850?fr=sidebar)

### 三色标记法

为了能让标记过程也能并行，Go 采用了三色标记 + 写屏障的机制。它的步骤大致如下：

1. GC 开始时，认为所有 object 都是**白色**，即垃圾。
2. 从 root 区开始遍历，被触达的 object 置成**灰色**。
3. 遍历所有灰色 object，将他们内部的引用变量置成 **灰色**，自身置成 **黑色**
4. 循环第 3 步，直到没有灰色 object 了，只剩下了黑白两种，白色的都是垃圾。
5. **对于黑色 object，如果在标记期间发生了写操作，写屏障会在真正赋值前将新对象标记为灰色**。
6. 标记过程中，`mallocgc` 新分配的 object，会先被标记成黑色再返回。

![img](https://upload-images.jianshu.io/upload_images/11662994-94548e98fe245de6.png?imageMogr2/auto-orient/strip|imageView2/2/w/741/format/webp)



还有一种情况，标记过程中，堆上的 object 被赋值给了一个**栈上指针**，导致这个 object 没有被标记到。**因为对栈上指针进行写入，写屏障是检测不到的**。下图展示了整个流程(其中 L 是栈上指针)：

![img](https:////upload-images.jianshu.io/upload_images/11662994-b2d93298df056f97.png?imageMogr2/auto-orient/strip|imageView2/2/w/758/format/webp)

为了解决这个问题，标记的最后阶段，还会回头重新扫描一下所有的栈空间，确保没有遗漏。而这个过程就需要启动 STW 了，否则并发场景会使上述场景反复重现。



### **流程**

1. 正常情况下，写操作就是正常的赋值。
2. GC 开始，开启写屏障等准备工作。开启写屏障等准备工作需要短暂的 STW。
3. Stack scan 阶段，从全局空间和 goroutine 栈空间上收集变量。
4. Mark 阶段，执行上述的三色标记法，直到没有灰色对象。
5. Mark termination 阶段，开启 STW，回头重新扫描 root 区域新变量，对他们进行标记。
6. Sweep 阶段，关闭 STW 和 写屏障，对白色对象进行清除。



### 混合写屏障

Go 在 1.8 版本引入了**混合写屏障**，其会在赋值前，对旧数据置灰，再视情况对新值进行置灰。大致如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/11662994-8994c6b1dc536940.png?imageMogr2/auto-orient/strip|imageView2/2/w/713/format/webp)

这样就不需要在最后回头重新扫描所有 Goroutine 的栈空间了，这使得整个 GC 过程 STW 几乎可以忽略不计了。



### 何时触发 GC

一般是当 Heap 上的内存达到一定数值后，会触发一次 GC，这个数值我们可以通过环境变量 `GOGC` 或者 `debug.SetGCPercent()` 设置，默认是 `100`，表示当内存增长 `100%` 执行一次 GC。如果当前堆内存使用了 `10MB`，那么等到它涨到 `20MB` 的时候就会触发 GC。

再就是每隔 2 分钟，如果期间内没有触发 GC，也会强制触发一次。

最后就是用户手动触发了，也就是调用 `runtime.GC()` 强制触发一次。

### 其他优化

扫描过程最多使用 25% 的 CPU 进行标记，这是为了尽可能降低 GC 过程对用户的影响。而如果 GC 未完成，下一轮 GC 又触发了，系统会等待上一轮 GC 结束。

对于 tiny 对象，标记阶段是直接标记成黑色了，没有灰色阶段。因为 tiny 对象是不存放引用类型数据（指针）的，这个在 [Go 语言内存管理（二）：Go 内存管理](https://www.jianshu.com/p/7405b4e11ee2) 提到过，没必要标记成灰色再检查一遍。



**GC分析**

[golang gc 优化思路以及实例分析_鼎铭的博客-CSDN博客](https://blog.csdn.net/weixin_42847874/article/details/103153349)

**GC优化**

1，函数尽量不要返回map， slice对象, 这种频繁调用的函数会给gc 带来压力。

2，小对象要合并。

3，函数频繁创建的简单的对象，直接返回对象，效果比返回指针效果要好。

4，避不开，能用sync.Pool 就用，虽然有人说1.10 后不推荐使用sync.Pool，但是压测来看，确实还是用效果，堆累计分配大小能减少一半以上。

5，类型转换要注意，官方用法消耗特别大，推荐使用雨痕的方式。

6，避免反复创建slice。

**GC 性能分析**

- go tool pprof
- go tool trace
- go build -gcflags=”-m”
- GODEBUG=”gctrace=1”



1. 





## Go RPC

[11 Go RPC - 简书 (jianshu.com)](https://www.jianshu.com/p/70cf3e3e4571)



## 打印 1~100

### 双携程

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	ch1 := make(chan struct{})
	ch2 := make(chan struct{})
	wg := sync.WaitGroup{}
	wg.Add(2)
	//线程A
	go func() {
		defer wg.Done()
		for i := 1; i <= 99; i += 2 {
			//等待 B 来唤醒
			<-ch2
			fmt.Println(i, "A")
			//唤醒 B
			ch1 <- struct{}{}
		}
	}()
	//线程B
	go func() {
		defer wg.Done()
		//首先唤醒 A
		ch2 <- struct{}{}
		for i := 2; i <= 100; i += 2 {
			//等待 A 来唤醒
			<-ch1
			fmt.Println(i, "B")
			//唤醒 A，执行到最后一次时不需要再唤醒
			if i != 100 {
				ch2 <- struct{}{}
			}
		}
	}()
	wg.Wait()
}
```



### 多协程打印 1~100

```go
package main

import (
	"fmt"
	"strconv"
)

var (
	target   = 100 // 要输出的最大值
	chCount  = 4
	curLine  = 0 // 通道发送计数器
	exit     = make(chan bool)
	channels []chan int // 协程切片
)

func main() {
	// 开启 n 个协程
	channels = make([]chan int, chCount)
	for i := 0; i < chCount; i++ {
		channels[i] = make(chan int)
	}

	// 多协程启动入口
	go ChanWork(channels[0])

	// 触发协程同步执行
	channels[0] <- 1

	// 执行结束
	if <-exit {
		return
	}
}

func ChanWork(c chan int) {
	for {
		// count为输出计数器
		if count := <-channels[curLine]; count <= target {
			fmt.Println("channel "+strconv.Itoa(curLine)+" -> ", count)
			count++

			// 下一个发送通道
			curLine++
			if curLine >= chCount {
				curLine = 0 //循环，防止索引越界
			}
			go ChanWork(channels[curLine])
			channels[curLine] <- count

		} else {
			// 通道编号问题处理
			id := 0
			if curLine == 0 {
				id = chCount - 1
			} else {
				id = curLine - 1
			}
			fmt.Println("在通道" + strconv.Itoa(id) + "执行完成")
			close(exit)
			return
		}
	}
}
```



## 多线程向两个账户中打款

### 通道

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	wg := sync.WaitGroup{}
	account := [2]int{}
	ch0 := make(chan int, 1)
	ch1 := make(chan int, 1)
	ch0 <- 0
	ch1 <- 1
	routineNumber := 1000
	wg.Add(routineNumber)
	for i := 0; i < routineNumber; i++ {
		go func() {
			defer wg.Done()
			select {
			case num := <-ch0:
				account[num]++
				ch0 <- num
			case num := <-ch1:
				account[num]++
				ch1 <- num
			}
		}()
	}
	wg.Wait()
	fmt.Println(account)
}
```

### 互斥锁

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	wg := sync.WaitGroup{}
	mutex0 := sync.Mutex{}
	mutex1 := sync.Mutex{}
	account := [2]int{}
	routineNumber := 1000
	wg.Add(routineNumber)
	for i := 0; i < routineNumber; i++ {
		go func(i int) {
			defer wg.Done()
			if i%2 == 0 {
				mutex0.Lock()
				account[0]++
				mutex0.Unlock()
			} else {
				mutex1.Lock()
				account[1]++
				mutex1.Unlock()
			}
		}(i)
	}
	wg.Wait()
	fmt.Println(account)
}
```



## 空结构

空结构（struct{}）是指没有字段的结构类型。他比较特殊，因为无论是其自身，还是作为数组元素类型，其长度都为零。

尽管没有分配数组内存，但是依然可以读写元素，对应的切片 len、cap 属性也都正常。

实际上，这类 “长度” 为零的对象通常都指向 runtime.zerobase 变量。

空结构可作为通道元素类型，用于事件通知。
