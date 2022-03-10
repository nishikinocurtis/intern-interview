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

**特点**

1.通道是有类型的，基于go的基本数据类型

2.通道是有方向的，可从通道读出数据，可向通道写入数据。当通道被传递时可设置其读写方向

3.通道是有缓存的，通过缓存容量可控制协程间的阻塞

4.通道是可关闭的，且只能被关闭一次，通道也是资源，如果不使用应关闭

**使用常识**

1.通道数据读写,通道写入数据后必须关闭；

2.从一个已经关闭的通道中读取数据,读完了之后,继续读会读到其通道类型的零值；

3.没有初始化的通道被关闭会报panic;

4.读取通道数据时应校验其有效性；

5.关闭通道时会产出一个广播，所有从通道读取数据的协程都会收到消息；

6.被遍历的通道如果没有收到通道被关闭的广播，遍历会一直被阻塞；

7.通道一般在写入协程处调用关闭，写只有一个写入协程的情况，通道被关闭后不能写入数据，但其他协程可以读出，注意重复关闭的情况；

**通道的读写和异常**

关闭通道的几个注意事项：

- 不能关闭一个没有初始化的通道
- 通道不能重复关闭
- 不能往已经关闭的通道中写入数据,但是可以从中读数据

**无缓存的通道**

> 使用一个无缓存的通道时应该注意，它是阻塞的：

- 没人读就永远写不进(阻塞)
- 没人写就永远读不出(阻塞)

**有缓存的通道**

可利用通道缓存能力进行协程调度，通道的元素个数或称缓存能力，决定协程是否产生阻塞，若通道数据已满则阻塞，写入阻塞读出也阻塞，这是相互的。

**select选择通道，协程多路复用**

select关键字是go特有的，其主要用于配合通道实现多路复用。

```go
func BaseChanner05() {
    //创建三个通道
    ch1 := make(chan int, 3)
    ch2 := make(chan int, 4)
    ch3 := make(chan int, 5)

    //创建3条协程
    go func(c chan int) {
        ticker := time.NewTicker(time.Second * 1)
        for {
            <-ticker.C
            c <- 1
        }
    }(ch1)
    go func(c chan int) {
        ticker := time.NewTicker(time.Second * 2)
        for {
            <-ticker.C
            c <- 2
        }
    }(ch2)
    go func(c chan int) {
        ticker := time.NewTicker(time.Second * 3)
        for {
            <-ticker.C
            c <- 3
        }
    }(ch3)

    //time.Sleep(time.Second)
    //主协程select多路复用,for不断获取不同通道的数据，随先来则优先处理谁。
    for {
        select {
        case chV1, ok := <-ch1:
            fmt.Printf("通道ch1输出：%v,有效性%v\n", chV1, ok)
        case chV2, ok := <-ch2:
            fmt.Printf("通道ch2输出：%v,有效性%v\n", chV2, ok)
        case chV3, ok := <-ch3:
            fmt.Printf("通道ch3输出：%v,有效性%v\n", chV3, ok)
    }
}
```

**通过容量控制并发数**

```go
func BaseChanner06() {
    //创建一个容量为5的通道，无论协程开多少，控制每次5条并发
    semaphore := make(chan int, 5)

    for i := 1; i <= 100; i++ {
        go func(c chan int, n int) {
            for {
                c <- i //抢通道写入,抢不到则阻塞
                fmt.Println("协程", n, "抢到通道")
                time.Sleep(time.Second)
                <-c //做完操作后自己读出,空出容量
            }
        }(semaphore, i)
    }

    for {
        time.Sleep(time.Second)
    }

}
```



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
- 重复以上知道所有的G执行完毕。



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



## 锁

Go的代码库中为开发人员提供了一下两种锁：

1. 互斥锁 sync.Mutex
2. 读写锁 sync.RWMutex

第一个互斥锁指的是在Go编程中，同一资源的锁定对各个协程是相互排斥的，当其中一个协程获取到该锁时，其它协程只能等待，直到这个获取锁的协程释放锁之后，其它的协程才能获取。

第二个读写锁依赖于互斥锁的实现，这个指的是当多个协程对某一个资源都是只读操作，那么多个协程可以获取该资源的读锁，并且互相不影响，但当有协程要修改该资源时就必须获取写锁，如果获取写锁时，已经有其它协程获取了读写或者写锁，那么此次获取失败，也就是说读写互斥，读读共享，写写互斥。

**实现原理**

Mutex实现中有两种模式，**1：正常模式，2：饥饿模式**，前者指的是当一个协程获取到锁时，后面的协程会排队(FIFO),释放锁时会唤醒最早排队的协程，这个协程会和正在CPU上运行的协程竞争锁，但是大概率会失败，为什么呢？因为你是刚被唤醒的，还没有获得CPU的使用权，而CPU正在执行的协程肯定比你有优势，如果这个被唤醒的协程竞争失败，并且超过了1ms，那么就会退回到后者(饥饿模式)，这种模式下，该协程在下次获取锁时直接得到,不存在竞争关系，本质是为了防止协程等待锁的时间太长。



## Go GC

[图解Golang的GC算法 - Go语言中文网 - Golang中文社区 (studygolang.com)](https://studygolang.com/articles/18850?fr=sidebar)

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



## Go RPC

[11 Go RPC - 简书 (jianshu.com)](https://www.jianshu.com/p/70cf3e3e4571)



## Go 双线程打印奇偶

```go
package main

import "fmt"

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)
	end := make(chan string)
    //线程A
	go func() {
		for i := 0; i < 100; i += 2 {
			//等待 B 来唤醒
			<-ch2
			fmt.Println(i, "A")
			//唤醒 B
			ch1 <- "print A"
		}
	}()
    //线程B
	go func() {
		//首先唤醒 A
		ch2 <- "begin"
		for i := 1; i < 99; i += 2 {
			//等待 A 来唤醒
			<-ch1
			fmt.Println(i, "B")
			//唤醒 A
			ch2 <- "print B"
		}
		end <- "end"
	}()
    
	<-end
}
```
