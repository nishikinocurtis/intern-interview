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
