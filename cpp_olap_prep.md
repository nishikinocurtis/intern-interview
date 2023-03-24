# 可能问题的收集(基于C++后端/OLAP引擎方向)

## 网络和Socket模型

- OSI Layering

	- 设计哲学? 功能解耦, 专注每一层的问题

	- 物理层: 无线/线缆

		- 蓝牙和WLAN有什么区别?

		- 光缆和网线呢?

		- 分时复用还是分频复用?

	- 数据链路层: 局域网传输

		- MAC是什么? Media-Access-Control 以什么单位区分? 网卡 有什么功能? 区分局域网内的硬件

	- 网络层: 广域网传输

		- ARP协议什么工作? IP转MAC, 完成实际分发 反向呢? RARP

		- 路由协议? BGP/OSPF/EIGRP, 区别?(概念上, 收敛速度上, 管理难度上)

		- ICMP: 提供错误信息用于debug(什么错误? 比如TTL过期), ping和traceroute使用ICMP

	- 传输层: TCP/UDP

		- TCP

			- 三次握手的过程: 发起方SYN: 进入SYN_SEND状态, 服务端收到, 进入SYN_RCV状态, 回复SYN/ACK, 发起方收到, 进入Established状态, 再次回复ACK, 服务端进入Established状态, 结束

			- 为什么不能两次? 服务端无法确认客户端接收能力/自己能不能成功发包, 

			- 第2次握手传回了ACK，为什么还要传回SYN？服务端传回发送端所发送的 ACK 是为了告诉客户端：“我接收到的信息确实就是你所发送的信号了”，这表明从客户端到服务端的通信是正常的。回传 SYN 则是为了建立并确认从服务端到客户端的通信。

			- 第三次能不能客户端带数据? 一般不带, 但源码没有阻止这种可能: 对于非阻塞socket, connect发起后立即写缓存, 导致connect成功时, write_buffer内有数据, 可能会发生ACK带数据

			- 四次挥手: 发起方FIN, 进入FIN-WAIT-1, 等待服务端ACK, 进入FIN-WAIT-2, 服务端进入CLOSE-WAIT. 后续服务端可以继续发消息, 客户端不再新发. 直到服务端结束, 发起FIN, 进入LAST-ACK. 客户端收到后进入TIME-WAIT, 等待两个MSL后关闭, 服务端则是收到最后一个ACK后关闭.

			- 为什么四次? 半关闭特性

			- 为什么等两个MSL? 一旦ACK失败, 对方会在1MSL后重传FIN, 重传的FIN到达客户端又需要一个MSL.

			- 重传: 一旦在一个RTO后依然没有收到ACK, 就重传. 如何确定RTO? 基于RTT的加权平均和方差, 太大不容易发生重传, 等待时间过长, 太短频繁重传, 容易阻塞.

			- 快速重传: ACK携带到目前位置最大的连续有序Seq No + 1. 也有问题: 对于多个丢包, 只知道最小的一次丢包.

			- 选择重传: 基于快速重传, 但分段. 一次丢包直接重传所属区间, 同时SACK报告已收到的其他分段, 可以快速缩小区间.

			- D-SACK: 标记重复包

			- 流控: 滑动窗口. 起源: 一个一个传太慢, 一次传太多会拥塞, 沟通一个合适的宽度. 两个窗口: 发送窗口和接收窗口, 收到ACK时发送窗口前移, 维护两个游标, 指向未确认的第一个包和要发送的第一个包, 窗口内没有要发送的包就暂停, 设置定时器和等待ACK. 收到数据包时接收窗口前移. 那么如何确定窗口大小? 主要由接受方根据自身缓存大小在ACK包中回复

			- Congestion control: 为了最大化利用网络负载(不浪费但也不拥堵)

				- 慢启动:

				- 拥塞避免

				- 拥塞发生

				- 快速恢复

			- Nagle优化

			- 粘包和拆包: 上层可以自行约定数据delimeter, 在发生粘包时可以分割, 拆包时可以等待其他包. 固定包大小(不flexible). 自行标记数据body长度, 接收时收齐位置.

		- UDP

			- 和TCP什么区别? Header更小, 不可靠(少很多可靠机制) -> 所以也更快, 适用与实时游戏和音视频会议, 对丢包比较宽容, 但对延迟不宽容；UDP不是点对点, 适合大量分发.

			- 用UDP设计可靠协议? 加入缺少的那些机制 (乱序, 丢包: SeqNo. 重传/冗余)
	- 应用层:

		- HTTP/HTTPS

			- 状态码: 1xx提示, 2xx成功, 3xx重定向, 4xx客户端问题, 5xx服务端问题.

				- 常见代码? 200 OK, 301 永久重定向, 400 Bad Request, 403 禁止访问, 404 Not Found, 500 内部错误, 502 网关错误

				- HTTP不同分段: 有`\r\n`隔开, 解决了上述TCP粘包拆包问题

				- 常见字段: Host, Content-length, Connection Keep-Alive(保持TCP连接)

				- 请求方法: 

				- 缓存技术: 强制缓存: 由服务器在Response头中标记Cache-Control或Expires实现, 客户端会保持缓存至这个时间, Cache-control 选项更多一些，设置更加精细，所以建议使用 Cache-Control 来实现强缓存；协商缓存: 第二次请求起带资源tag, 交给服务器比较, 如果服务器认为不需要重传数据, 就响应304. 具体实现方式?

				- 版本特性: HTTP1.0没有长连接, HTTP1.1引入长连接和Pipeline, 但会导致队头阻塞, 2.0引入多路复用, 3.0改用QUIC/UDP.

				- HTTPS: 在HTTP下加入TLS层, 非对称+对称混合加密 (RSA太慢, 直传AES不安全)

					- 中间人攻击: 证书和数字签名, 验证过程, 

		- DNS

			- 请求过程: Local DNS Server -> Root Server -> gTLD Server -> NameServer

			- 请求策略: 迭代 or 递归?

			- 常用端口? UDP53

			- linux命令: nslookup, dig, nscd restart

		- RPC

			- 和HTTP区别? 其实RPC也可以基于HTTP协议实现, 更多的是开发, 迭代和治理思维的变化: 传输效率: Thirft/Protobuf等都是二进制传输, 体积小效率高, 而非JSON; RPC框架一般自带服务注册/发现等治理模块, 和负载均衡等. 缺点: 需要预定义scheme等, 不适合对外暴露

- IO多路复用

		- IO模型: 同步阻塞BIO: 单线程(recv/send)会阻塞其他请求, BIO多线程: 每个请求一个线程, 会很快膨胀, 超过系统能力. 同步非阻塞NIO: accept请求后加入一个set, 每次轮询处理数据, 浪费cpu.

		- 多路复用: 每个线程都可以找到多个有事件的socket fd处理, select, poll(select + larger fd set), epoll(event poll: not scanning fd but register fd with call back). 详细讲讲epoll: 基于事件回调: 请求进来时直接查询谁在听这个事件, 而不是一个个扫描发生了什么事件.

		- epoll的模式: LT/ET模式: LT只要buffer有剩余数据可读就会触发epoll_wait, ET只在首次触发, 注意把buffer读干净.

		- duplicate fd问题: 先把fd1关了, 没有取消注册, 导致无法从epoll中取消注册fd1(因为fd1无效了), 后续fd2继续听事件, 还会一直触发fd1, 却无法操作. 解决方案: 先取消注册, 再关闭fd

		- 惊群问题: 父子共享fd: 导致同时唤醒；多个线程注册同一个fd, 也导致同时唤醒. 对于父子问题: 因为还是同一个epoll实例, 可以用ET模式保证只通知一次, 但对于第二个问题, 可以用Linux新特性, `EPOLLEXCLUSIVE`参数. 但还是有问题:

		- 无效唤醒, 饥饿问题...


- 网络统一是大端的: 最重要的bit最先出现. 转换: htonx, ntohx.

- Socket

	- 状态流程: Server: create, bind, listen, accept；Client: create, bind, connect


- 高并发处理

	服务侧缓存(Redis), 用户侧缓存(CDN), 数据库性能优化, 短期:削峰(MQ), 长期:扩容(负载均衡+resharding+分库分表+定期落盘+服务注册/发现)

	- CDN: 适用范围? 静态内容, 实际问题? 智能分配, 防盗链

	- 其他: 下面再说

- zerocopy

	- mmap + write

	- sendfile

	- DMA收集拷贝 + sendfile




## 进程和线程: 原理,通信,同步,异常处理

- 原理和概念:

	- Process: 操作系统管理大部分资源的最小单元: 地址空间

	- Thread: 轻量级的调度单元, 通过Share地址空间实现concurrency, 有局部栈/PC/Register, 但也有内核/用户/混合中的thread实现

	- Coroutine: Work on threads, 完全由协程库(用户态处理) (Pros&Cons: 减少内核切换, 但一旦Process不在运行所有coroutine都不能运行)

- 通信:

	- 进程: pipe(有亲缘关系)(内存only)和named pipe(无亲缘关系)(有fs/磁盘节点), 共享内存(需要上锁来保证线程安全, 最快!)/信号量, Signal, MQ(在内核中, 可以有格式), 网络(rpc/socket)

	- 线程呢? 线程共享地址空间, 没有通信必要, 只要做好线程安全.

	- 速度, 容量和expressiveness呢?

3. 同步:

	锁(mutex, spin), 信号量(semaphore), condition variable, 原子操作

	- Deadlock and avoidance

		- 四个条件: 互斥, 占有等待, 非抢占, 依赖环

		- 解决方法: 预防, 避免, 检测+解除, 银行家算法

	- S-X Lock: Also CPU缓存一致性


	- 一些规范

	- 生产/消费者

	有什么性能问题? 安全问题?

	- 锁分类:

		- mutex: 失败后释放(有context switching成本)

		- spin: 失败后busy waiting(由CPU的原子命令不断查询)

		- 读写锁: S-X Lock, 读不阻塞, 队列中有写就会阻塞后面的读请求

		- 乐观锁/悲观锁: 乐观是无锁操作, 先操作, 有更新就放弃, 悲观是每次都上锁

4. 异常处理, 中断和信号

5. 线程池

就是复用线程, 不再重复创建和销毁

参数: 取决于任务类型, 越偏向IO密集的可以给越多quota, 避免阻塞浪费CPU时间


## 分布式

- 时钟同步

- 下面的一致性问题

- 一致性快照

- 2阶段提交, 出口提交



## 内存管理,分区

- 管理什么? 分配释放, 追踪

- 怎么管理? Contiuous, or Block, page, segment, page_segment

1. 虚拟内存, 分区, MMU



- 分页, 分段, 页表, 多级页表 TLB重填

	- 页表加速: 从vpage到物理page的映射表, 但每次都去内存中搜索比较慢, 所以有了TLB.

	- 多级页表: 页表全部存在内存中比较占空间, 通过分级提高管理粒度, 每次load一小部分

	- CPU寻址: MMU

	- 页面置换(FIFO, LRU, LFU, Clock, working set), 分段置换, 局部性

	性能问题? 

- 内存对齐, 栈溢出

	- pragma pack自动对齐, 宽度标记手动对齐

- fork: copy on write:

	Linux中fork时内存空间发生了以下变化：

	fork函数从已存在的进程中创建一个新的进程，新进程为子进程，原进程为父进程。
	系统先给子进程分配资源，例如存储数据和代码的空间，然后把父进程的所有值都复制到子进程中，只有少数值与父进程的值不同。
	子进程复制了父进程的task_struct结构和系统堆栈空间，但其他资源却是与父进程共享的，比如文件指针，socket描述符等。
	子进程被创建后，父进程的全局变量和静态变量复制到子进程的地址空间中，这些变量将相互独立。
	fork时使用相同的物理内存，并通过写时复制（copy-on-write）的方式维护各自的数据。

	Copy-on-write (COW) in Linux memory management is mainly used to share the virtual memory of operating system processes, in the implementation of the fork system call12. When a process forks a child process, instead of copying all the data in the parent process to the child process, it marks certain pages of memory as read-only and keeps a count of the number of references to each page34. If either process tries to modify a shared page, a page fault occurs and the kernel will copy that page before allowing the write15. This way, COW can speed up the creation of child processes and save memory space32.



## 调度

进程调度算法/LRU: 优缺点? 进化过程?

上下文切换过程: 保存什么? 保存在哪? 如何回复? 有异常怎么办

## OS其他

特权机制: 优缺点?

	系统调用

文件系统

函数调用过程: Caller and callee-save, 栈帧, ABI

## 数据库: MySQL

1. 引擎

	MyISAM, InnoDB和InMemory对比: 
	InnoDB保证可重复读, 无性能损失, 支持事务, 支持行锁, 支持外键(但不建议使用)
	支持redo log自动恢复, 支持MVCC, 性能更强(MyISAM不支持并发)

	MERGE/ARCHIVE

2. 并发控制/一致性问题

	- 可能出现的问题: R-R, W-R, R-W, W-W

	- 隔离级别

		4 level: read-uncommitted, read-committed, repeatable-read, serielizable.

		脏读, 幻读, 不可重复读: 脏读是指一个事务在处理数据的过程中，读取到另一个未提交事务的数据；不可重复读是指对于数据库中的某个数据，一个事务范围内的多次查询却返回了不同的结果，这是由于在查询过程中，数据被另外一个事务修改并提交了；指一个事务执行两次查询，但第二次查询的结果包含了第一次查询中未出现的数据。

	- 事务性是什么(ACID), 实现方式, MVCC

		- redo log: D

		- undo log: A, MVCC

		- MVCC保证了什么: 一致性非锁定读；如何实现:增加事务id, 回滚指针, 隐藏row id

		- 一致性锁定读: select for update, select lock in share mode, 可以在读取时加S/X锁.

	- lock

		- 表锁, 行锁, gap lock, next key lock(row + gap)(record lock只能锁已经存在的, 对于范围查询, 需要同时锁范围避免并发更新影响结果)

		- intention lock: 锁表

		- auto-inc lock

3. 索引

	一种利用有序性快速查询的数据结构

	- 底层实现: Hash表(加挂链/挂红黑树), 但不支持范围查询；B/B+树 区别?(B+相邻节点有链表, 方便范围查询, 数据都在叶子节点, 复杂度稳定)

	- Clustered / Non-clustered? 数据本身和索引结构是否在一起

	- Primary / Unique

	- 覆盖(Index contains data, 无需回表), 联合(Multi column), 全文(需要分词, etc. 不如ES)

	- 最左前缀: 从左向右一次filter, 所以能筛掉更多data的column放前面.

	- 索引优化: NOT NULL, for frequent read, for condition, for order, for join, not for frequent write, not too much, union preferred than multiple single indices, avoid redundance, use prefix index for string

	- explain查看执行计划

	- 索引失效: select\*, 组合索引查询顺序, 在索引列上做了转换, like%查询, or的条件中有非索引列, 类型implicit转换

4. 日志

	- redo log: encodes request to change table data, recover unfinished modifications (未物理完成事务的恢复)

	- undo log: each associates with a read/write operation, for other transactions to get history / unmodified data. (并发多版本冲突的解决/失败事务的回滚) 解决原子性: 没提交的就对其他并行事务不生效.

	- binary log: for replication & recovery, only used for statements modifying data. resilient to unexpected halts, only completed events are logged. (数据主体的恢复), 不是由引擎提供.

	- slow query log: mysqldumpslow command, record queries that take long time or examine many rows, for optimization

4. 范式

    - 1,2,3,3.5,BC-NF

5. 优化

	



## 数据库: Redis


1. SQL和NoSQL, 或者说: RDB和NRDB

2. Hash问题

3. 缓存穿透

4. 跳表



## 一致性级别:

Causal, Linearizable, Sequential

## 设计模式

单例模式(Tracing Exporter)

适配器模式(Abstraction)

工厂模式(通用接口, 无需指定具体类, 可以由参数自动确定)


Java的一些: 依赖注入, 控制反转, 切面编程

## Linux

命令相关

shell算术

grep

find

kill

ss

ip

ps

nm

netstat

traceroute

cp, rn, mv


## C++:系统相关

`volatile`关键字: 当两个线程都要用到某一个变量且该变量的值会被改变时，应该用 volatile 声明，该关键字的作用是防止优化编译器把变量从内存装入 CPU 寄存器中。 如果变量被装入寄存器，那么两个线程有可能一个使用内存中的变量，一个使用寄存器中的变量，这会造成程序的错误执行。

环境感知: 怎么知道当前系统架构? 

内存分区

断点调试(gdb问题)

## C++:多线程

C++11线程库:

原子

异常处理中的线程等待

全局变量进行异常捕获

## C++:面向对象

- `final`关键字 (C++11, 阻止)

	根据搜索结果123，final说明符有以下作用和用法：

	- 用于防止类的进一步派生或虚函数的进一步重写。
	- 可以修饰非抽象类、非抽象类成员方法和变量。
	- 只能放在类名或函数声明的后面。
	- 不能修饰纯虚函数，因为没有意义。

- `explicit`构造
	
	explicit is a keyword used before constructors and is defined as making the constructor not conduct any implicit conversion by specifying the keyword explicit. 
	This means that a constructor or conversion function (since C++11) or deduction guide (since C++17) marked with explicit cannot be used for implicit conversions and copy-initialization.
	For example, if you have a class A with an explicit constructor that takes an int argument, you cannot write A a = 5; because that would imply an implicit conversion from int to A. You have to write A a(5); or A a = A(5); instead.

- `delete`构造删除

- `default`构造

- `virtual`方法

- `abstract`类

- `friend`

继承方式

泛型编程

内存布局(类的), 对齐, 大小

Text, data, bss, heap go up, stack go down, highest arguments

默认栈空间大小: 8Me

- 多态原理: 虚表和虚指针

	静态多态是通过重载和模板技术实现的，在编译期间确定；动态多态是通过虚函数和继承关系实现的，执行动态绑定，在运行期间确定

	当编译器发现类中有虚函数时，会创建一张虚函数表，把虚函数的函数入口地址放到虚函数表中，并且在对象中增加一个指针vptr，用于指向类的虚函数表。当派生类覆盖基类的虚函数时，会将虚函数表中对应的指针进行替换，从而调用派生类中覆盖后的虚函数，从而实现动态绑定

	定义纯虚函数是为了实现一个接口，起到规范的作用，想要继承这个类就必须覆盖该函数, 实现方式是在虚函数声明的结尾加上= 0即可。
	
	虚函数表是针对类的，类的所有对象共享这个类的虚函数表，因为每个对象内部都保存一个指向该类虚函数表的指针vptr，每个对象的vptr的存放地址都不同，但都指向同一虚函数表。

	构造和析构: 构造一个对象的时候，必须知道对象的实际类型，而虚函数是在运行期间确定实际类型的。如果构造函数为虚函数，则在构造一个对象时，由于对象还未构造成功，编译器还无法知道对象的实际类型，是该类本身还是派生类。无法确定. 虚函数的执行依赖于虚函数表，而虚函数表是在构造函数中初始化的，即初始化vptr，让它指向虚函数表。如果构造函数为虚函数，则在构造对象期间，虚函数表还没有被初始化，将无法进行. 如果析构函数不被声明成虚函数，则编译器实施静态绑定，在删除基类指针时，只会调用基类的析构函数而不调用派生类析构函数，这样就会造成派生类对象析构不完全。所以，将析构函数声明为虚函数是十分必要的。


## C++:语言特性

`noexcept`关键字

Iterator

Dangling指针和wild指针

vector原理, 扩容, 均摊复杂度

运算符重载

## C++:新特性

- L-Value vs. R-Value:

可以被取地址的是左值，表达式、不可取地址的是右值

- R-Value Reference

由`&&`标记，可以引用表达式的值，延长其在内存中的生命周期，避免重复拷贝。
由std::move创建, 原对象的值失效.

- 智能指针：

传统指针: 手动申请, 手动释放

`auto_ptr`: 使用copy constructor实现move, 随着move语义的引入而失效

`shared_ptr`和`unique_ptr`, `weak_ptr`解决循环引用问题, 或者观察者模式, 不需要管理生命周期

```cpp
shared_from_this()
```

- 类型转换

- lambda函数

- auto遍历器

- 函数类型推断

## 数值计算

## 编译原理

## 体系结构

## 数据结构和算法

Bitmap

快选

KMP

AC自动机

DP

## 分布式追踪:Dapper论文

采样类型

如何解决罕见因素

基于query

## 数据库优化: LSM树, B/B+树, 红黑树对比和原理

结构? 性能? 复杂度?

一切的根源都是性能:

快速范围查询, 总体查询复杂度, 同等复杂度的常数, 文件和内存读写次数...


## 分布式共识:Raft论文和其他原则

- 运作过程


- 容错能力


- 实现细节



## 大数据相关

1. Docker

	- 命令

	- 原理

	- 问题

2. Istio, k8s

	- 集群调度


	- 容错


	- 扩展


	- 问题

3. Hadoop生态

### OLAP相关


### 性能优化:机制和实现

### 容错:机制和实现

## 问的多的(SOTA的): RocksDB

## 问的多的(SOTA的): ClickHouse

## 问的多的(SOTA的): HBase/Hive

