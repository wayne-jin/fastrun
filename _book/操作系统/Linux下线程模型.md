# Linux下线程模型

## **基础概念**

线程是操作系统能够调度和执行的基本单位，在Linux中也被称之为轻量级进程。从定义中可以看出，线程它是操作系统的概念，在不同的操作系统中的实现是不同的，不过今天分享的猪脚是Linux Thread。

对于Linux操作系统而言，它对Thread的实现方式比较特殊。在[Linux内核](https://www.zhihu.com/search?q=Linux内核&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})中，其实是没有线程的概念的，它把所有的线程当做标准的进程来实现，也就是说Linux内核，并没有为线程提供任何特殊的调度语义，也没有为线程实现特定的数据结构。取而代之的是，线程只是一个与其他进程共享某些资源的进程。每一个线程拥有一个唯一的task_struct结构，Linux内核它仅仅把线程当做一个正常的进程，或者说是轻量级进程，LWP(Lightweight processes)。

对于其他的操作系统而言，比如windows，线程相对于进程，只是一个提供了更加轻量、快速执行单元的抽象概念。对于Linux而言，线程只是进程间共享资源的一种方式，非常轻量。举个简单例子，假设有一个进程包含了N个线程。对于那些显示支持线程的操作系统而言，应该是存在一个[进程描述符](https://www.zhihu.com/search?q=进程描述符&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})，依次轮流指向N个线程。这个进程描述符指明共享资源，包括内存空间和打开的文件，然后线程描述它们自己独享的资源。相反的是在Linux中，只有N个进程，因此有N个task_struct数据结构，只是这些数据结构的某些资源项是共享的。

这里再总结一下，Linux线程是进程资源共享的一种方式，而其他操作系统，线程则是一种实现轻量、快速执行单元的抽象概念或者实体。这里再深入的理解一下，Linux中的线程和进程的区别。这也是诸多面试题中，最常见的一个。

## **资源共享**

Linux线程与进程的区别，主要体现在资源共享、调度、性能几个方面，首先看一下资源共享方面。上面也提到，线程其实是共享了某一个进程的资源，这些资源包括：

- 内存地址空间
- 进程基础信息
- 大部分数据
- 打开的文件
- 信号处理
- 当前工作目录
- 用户和用户组属性
- 等等

哪些是线程独自拥有的呢？

- 线程ID
- 一系列的[寄存器](https://www.zhihu.com/search?q=寄存器&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})
- 栈的局部变量和返回地址
- 错误码 errno
- [信号掩码](https://www.zhihu.com/search?q=信号掩码&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})
- 优先级
- 等等

这里说一个黑科技，线程拥有独立的调用栈，除了栈之外共享了其他所有的段segment。但是由于线程间共享了内存，也就是说一个线程，理论上是可以访问到其他线程的调用栈的，可以用一个[指针变量](https://www.zhihu.com/search?q=指针变量&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})，去访问其他线程的局部栈帧，以访问其他线程的[局部变量](https://www.zhihu.com/search?q=局部变量&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})。

## **调度**

说到调度，就得提到进程的上下文切换。上下文切换也被称作为进程调度或者任务切换，简单的来说是把CPU从一个进程或者线程切换到另一个执行。概括的来说，线程的上下文切换，要比进程更加快速，因为本质上，线程很多资源都是共享进程的，所以切换时，需要保存和切换的项是很少的。

线程上线文切换时，虚拟地址空间是不变的，但是进程上下文切换时，是需要重新映射虚拟地址空间。进程切换上下文时，进出OS内核&寄存器切换，是最大的时间支出。更模糊的代价是上下文切换时，会干扰处理器的缓存机制。当上下文切换时，处理器需要重新cache一些内存。

这里更大的一个区别时，当更改虚拟地址空间时，CPU 的 TLB 等也会被刷新，导致接下来的内存访问更加耗时，所以相对线程切换来说，进程的切换耗时更大。

## **性能**

从性能方面，来查看一下线程与进程的对比。由于线程更加轻量，导致线程的创建速度、切换速度都要高于进程。这里就有一个疑问了，从上面提到的各个方面来看，好像线程都要优于进程，那么有没有啥缺点呢？

## **线程缺点**

线程同样也有缺点，最大的缺点是线程的不安全性，缺乏保护机制。就是上面提到的黑科技，因为线程间共享数据，一个线程可以重写另外一个线程的堆栈，导致出现一些异常的情况。除此之外，线程还有以下缺点：

- 共享属性：[全局变量](https://www.zhihu.com/search?q=全局变量&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})是在所有线程间共享的，访问时是需要同步加锁。
- 很多库函数是线程非安全的，[多线程](https://www.zhihu.com/search?q=多线程&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})编程时，需要注意这一点。
- 线程的健壮性不强，如果一个线程crash了，那么整个应用程序就跪了。

## **应用场景**

上面提到了线程与进程的对比，也提到了线程的优点和缺点，那么什么情况下适合用线程呢？简单的来说，计算密集型的任务，适合于多线程来处理。因为计算密集型任务，需要耗费很多CPU，上下文的切换是非常频繁的，而线程切换速度是高于进程的，所以使用线程是更加适合的。在实际的编程过程中，根据业务的场景，再结合进程和线程的优缺点对比，来决定适合的编程模型。

## **线程创建**

那么Linux中线程是如何创建出来的呢？上面也提到，在Linux中线程是一种资源共享的方式，可以在创建进程的时候，指定某些资源是从其他进程共享的，从而在概念上创建了一个线程。在Linux中，可以通过clone系统调用来创建一个进程，它的函数签名如下：

```text
#include <sched.h>
int clone(int (*fn)(void *), void *child_stack, int flags, void *arg, ...);
```

我们在使用clone创建进程的过程中，可以指明相应的参数，来决定共享某些资源，比如:

```text
clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0);
```

这个clone系统调用的行为类似于fork，不过新创建出来的进程，它的内存地址、文件系统资源、打开的[文件描述符](https://www.zhihu.com/search?q=文件描述符&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})和信号处理器，都是共享父进程的。换句话说，这个新创建出来的进程，也被叫做Linux Thread。从这个例子中，也可以看出Linux中，线程其实是进程实现资源共享的一种方式。

## **[内核线程](https://www.zhihu.com/search?q=内核线程&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})**

在Linux中，还存在一个Kernel Thread的概念，也就是内核线程。内核创建一些内核线程来执行一些后台任务。相对于普通的进程，内核线程完整的存在于内核空间，是没有自己的地址空间的，也就是mm指针为空，它的操作仅存在于内核态，并且也不会上下文切换到用户态。不过内核线程和普通进程类似的是，是可调度和可抢占的。

## **同步**

由于线程间共享了很多资源，所以在多线程的编程环境下，为了保障结果的准确性和一致性，需要对共享资源的访问进行同步。常见的同步方式，也就是加锁，以保障操作共享资源时，不会出错。在Linux中，锁的种类大致有四种:

- 互斥锁
- 读写锁
- 条件变量
- 自旋锁
- [内存屏障](https://www.zhihu.com/search?q=内存屏障&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A57349087})

有兴趣的同学，看看看下这篇文章：[http://blog.lecury.cn/2016/02/21/%E5%90%8C%E6%AD%A5%E4%BA%92%E6%96%A5(%E9%94%81).html](https://link.zhihu.com/?target=http%3A//blog.lecury.cn/2016/02/21/%E5%90%8C%E6%AD%A5%E4%BA%92%E6%96%A5(%E9%94%81).html) 。总结来说，锁的代价是高昂的，所以在设计高并发、高吞吐的程序时，尽量避免锁的使用，或者减少锁的区间。

## **常见的多线程编程模式**

下面谈一下实际工作中，要如何合理的线程呢？这里我简单的提出三种常见的线程模型。

- leader-follow 模型（主从）

- - 线程与连接对应，并发度等于线程数。
  - 所有线程经历accept->close整个过程。
  - 适用于连接数少、处理时间长、CPU密集型服务。

- producer-consumer模型（生产者消费者）

- - 主线程用于accept请求，并将fd放置在消费队列pendingpool中。
  - pendingpool进行连接的维护工作。
  - 多个worker竞争pendingpool的连接。
  - 适用于连接数多、处理速度快的业务。

- 高并发索引模型

- - 无锁设计
  - 将请求或者事务映射到具体线程处理

## **踩过的坑和小技巧**

- 同步
- 过载保护
- 公平调度
- 析构出core
