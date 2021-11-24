## GDB的使用

### 编译选项

要调试`C/C++`的程序，在编译时，要把调试信息加到可执行文件中。编译器(cc/gcc/g++)添加`-g`参数。

比如：g++ -g hello.cpp -o hello

### 启动gdb

\1) gdb -p 进程ID

```
调试正在运行的程序, 找到进程ID之后, gdb -p过去就可以
```

\2) gdb 二进制文件名

```
直接用gdb启动程序, 如果程序做过fork之类的, 需要设置gdb参数`set follow-fork-mode child`
```

\3) gdb 二进制文件名 coredump

```
调试dump文件
```

### 常用的调试命令

#### 设置断点：

\1) b 文件名:行号

```gdb
   (gdb) b main.cpp:10
```

\2) b 函数名

````
```gdb
(gdb) b ProcessMessage
(gdb) b Server::Loop
```
````

#### 条件断点:

cond(ition) bnum expression

例如想要在a变量等于1的时候, 进入断点10, 那么就是:

```gdb
   (gdb)cond 10 a == 1
```

#### 数据断点

watch 变量

例如有一个变量a, 我想知道他在那里被改变了

```gdb
   (gdb)watch a
```

也有可能是程序写越界导致了a变量被改变(a的类型为int)

```gdb
   (gdb)watch *(int*)&a
```

#### 启用关闭断点

info b可以看到所有的断点信息(状态)

enable 断点编号, 启用断点

disable 断点编号, 关闭断点

delete 断点编号, 删除断点

```gdb
   (gdb) info b
   (gdb) enable 1
   (gdb) disable 2
   (gdb) delete 3
```

#### 恢复程序运行和单步调试：

step/s : 单步跟踪，如果有函数调用，会进入该函数

next/n : 单步跟踪，如果有函数调用，不会进入该函数

finish : 运行程序，知道当前函数完成返回

continue/c : 继续运行

run/r : 运行程序

#### 其他常用命令：

p i : 打印变量i的值，p是print命令的缩写

x : 用来查看一段内存的指令, x/nfu, n是个数, f是format, u是单位(b,h,w,g), 具体需要看gdb手册

bt : 查看函数堆栈

f: 切换函数栈, 比如f 1, f 10

frame 栈帧号 : 打印栈帧

list : 显示程序中的代码, 可以通过+-来控制向上翻还是向下翻

quit ：退出gdb

#### 设置变量的值

有时候需要在运行的时候修改变量的值, 可以通过set来做

```gdb
   (gdb) set a = 10
   (gdb) set *(int*)0x8048a51 = 100
```

### coredump等问题的定位及解决

#### gdb coredump应用程序 coredump文件

#### 通过以下命令调试

info threads 显示所有线程

bt 显示线程堆栈信息

thread thread_num 切换线程

frame num 切换栈

### gdb常见问题

\1) C++11的auto变量打印

```
auto变量可能不能正常打印期数据和类型
```

\2) 捕获信号

发送给程序的信号, 可能会被gdb捕获, 而不是被程序处理, 这是正常现象 3) 变量值错误

gdb有时候会打印错误的变量值, 跟log的值不一样, 推荐尽量打log, log里面的值一定是正确的, gdb里面不一定 4) 复杂类型变量

gdb支持`pretty printer`, 可以让复杂类型看起来更舒服一些

```gdb
   (gdb) set print pretty on
   (gdb) p *this
   $2 = {
    <gsdk::Singleton<gsdk::message::MainLoop>> = {
        <gsdk::Noncopyable> = {<No data fields>}, 
        members of gsdk::Singleton<gsdk::message::MainLoop>:
        static once_ = {
            _M_once = 2
        }, 
        static ptr_ = 0x612000000ac0
    }, 
    members of gsdk::message::MainLoop: 
    run_ = true,
```

gdb可以通过python插件扩展来支持个复杂类型的打印, 例如STL等, Ubuntu等发行版自带了STL的pretty printer.

```gdb
   (gdb) p this->queue_.queue_ 
   $4 = std::vector of length 0, capacity 1
```

  GDB使用及问题排查

### 1. GDB基本操作

#### (1) 开启GDB调试

- 调试可执行文件program，一般直接在需要调试的文件目录下执行。注意，待调试的文件需要使用-g参数编译才能使用GDB调试。

  $gdb program

  `gdb gateway`

- **常用操作**

  r(run) 开始调试，会自动跳到第一个断点，若没有断点，直到执行完代码或错误停止。

  l(list) 查看当前执行代码上下文，方便调试过程中查看代码。

  n(next) 单步调试，执行下一句代码，若下一句代码为函数调用，不进入函数内部执行。

  s(step) 进入函数内部调试。

  c(continue) 继续执行，直到下一个断点或错误停止。

  p(print) x 打印出x的当前值。

  until l 直接运行至指定行数l。

  f(finish) 运行程序直到程序结束。

  q(quit) 退出gdb。

- **断点相关操作**

  info break 查看当前所有断点的信息。

  b(break) 13 在第13行设置一个断点。

  b main 在main函数入口处设置一个断点。

  break server.cpp:25 在当前文件夹下其他文件server.cpp中的第25行设置断点。

  break sdk/net/ws.cpp:28 在当前项目其他文件ws.cpp中的第28行设置断点。

  delete 5删除编号为5的断点。

  disable/enable 25 禁用/启用编号为25的断点。

  break 15(where) if i==3(condition) 假设第15行代码为for(int i = 0; i < 10; ++i)，当执行该语句且i==3时，此处断点生效。

在实际项目中，需要多次调试同一个文件，因此保留每次调试的断点设置是很有必要的，gdb中也提供这样的功能，能够方便的存储和读取断点。

\1) 保存断点

先用info b 查看一下目前设置的断点，使用save breakpoint命令保存到指定的文件，这里以进程名字+bp后缀为文件名，输入命令

```
save breakpoint gateway.bp
```

![image-20211122134546480](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134546480.png)

\2) 读取断点

注意，在gdb已经加载进程的过程中，是不能够读取断点文件的，必须在gdb加载文件的命令中指定断点文件，具体操作是使用-x参数。例如，我需要调试gateway_server这个文件，同时使用上次存储的断点文件gateway.bp。输入命令

```
gdb gateway_server -x gateway.bp
```

![image-20211122134619871](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134619871.png)

- **多线程调试**

info thread 查看当前进程的线程。

thread 切换调试的线程为指定ID的线程。

#### (2) GDB查看core文件，定位问题代码

core是程序非法执行后core dump后产生的文件。测试源代码如下图，strcpy将指向一个常量字符串的指针拷贝给了一个NULL指针，肯定会导致段错误的发生，但有时候core文件并不是默认产生的，可以通过ulimit –a检查是否设置了core文件的生成大小。

![image-20211122134643837](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134643837.png)

![image-20211122134710639](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134710639.png)

当core file size为0时，意味着不会生成相关文件，需要通过ulimit –c 参数进行设置。输入指令

```
ulimit –c 1024
```

![image-20211122134738932](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134738932.png)

此时再运行生成文件，即可生成对应的core文件。

![image-20211122134814257](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134814257.png)

查看core文件

$gdb

```
$gdb test core
```

![image-20211122134848897](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134848897.png)

可以看到core的产生是由于strcpy的错误使用导致的，由此定位到了错误的源头。

#### (3) Watch观测变量，发现逻辑问题

gdb调试中使用watch观测，可以观测变量值的变化，当被监测变量值发生变化时，程序会被断点暂停，可通过该方法来追踪定位一些难以发现的逻辑错误。在下图的例子中，由于sizeof()取值大于变量b的实际大小，导致memset错误地修改了变量a的值。

![image-20211122134920231](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134920231.png)

在gdb调试时，可通过指令watch来指定观测目标a的变化，也可以通过地址来观测。

![image-20211122134953734](C:\Users\jinjiawei\AppData\Roaming\Typora\typora-user-images\image-20211122134953734.png)

当watch观测的变量值发生了变化时，gdb会输出该变量变化前后的取值，这样即可发现执行了第6行代码后，a的取值发生了变化，由此定位到错误发生在第6行代码处。

### 2. 程序常见问题分析

在Linux编程中，遇到程序出错是在所难免的，主要分成程序崩溃和程序卡死两类错误。程序崩溃最常见的错误就是程序运行终止----Segmentation fault (core dumped)，解决此类问题，主要可以从内存访问错误、资源访问冲突以及内存崩溃及溢出的方向进行排查。而程序卡死的错误相对而言排查起来容易一些，主要是程序的逻辑错误导致的死循环，或是多线程开发中的死锁的情况。

#### (1) 程序崩溃

Segmentation fault导致的程序运行终止，其根源是在于对内存的错误读写操作。此时的段错误，就是指访问的内存超出了系统为当前进程分配的内存空间，一旦某进程发生了越界访问，CPU会产生相应的异常保护，因此报出了Segmentation fault。导致段错误的原因主要分以下几类来说明，在实际生产环境碰到程序崩溃时，即可分点进行排查。

- **访问非法内存地址**

有些内存地址是由内核占用并保留的、有些内存正在被其他程序正在使用，为了保证系统正常工作，所以会受到系统的保护，而不能任意访问；访问地址为0的空指针同样会导致内存访问错误。因此在使用指针的过程中，一定要注意指针的初始化与强制类型转换过程。

错误代码如下：

```
void invalid_ptr{  
long *ptr;
    *ptr = 0;                         // 空指针
    ptr = (long *)0x12345678;
    *ptr = 100;                       // 非法地址访问
}
```

- **内存访问越界**

简单的说，代码中主动申请了一块内存，但在实际使用过程中，超出了申请的内存大小，会破坏其他内存区域的数据。

内存访问越界不一定会在第一时间报错，因为可能写入了一块可用的内存区域，但这种情况只是暂时的，该内存对系统而言仍是可分配对象，程序可能面临更大的风险，更糟糕的是在栈内存发生的内存访问越界错误。

错误代码如下：

```
void out_of_mem{
    int data[2];
    data[3] = 0;
}
```

由于此时内存访问越界，栈内存中的数据已经遭到了破坏，程序无法从栈内存获取到正确的返回地址数据，或者被破坏后的地址正好指向另一个程序，最终导致程序崩溃。一些病毒程序就是通过有意识的内存越界，覆盖栈中的函数返回地址，从而半路拦截程序去执行恶意代码。

- **访问共享资源冲突**

在多线程编程中，常常会出现不同线程对同一数据资源进行读写操作，在不做处理的情况下，任由程序执行，容易造成资源访问错误。

错误代码如下：

```
char * ptr = (char *)malloc(sizeof(char));    // 全局变量
if (ptr){                        // 线程1
    *ptr = ‘a’;
}
free(ptr);                    // 线程2
ptr = NULL;
```

在这个错误案例中，先定义了一个全局变量char *ptr，线程1与线程2均可对ptr进行读写操作，但由于多线程程序是并行执行的，线程1与线程2在对ptr进行IO操作时可能会发生冲突。比如，线程2先通过free释放了ptr申请的内存，但还没来得及将ptr设置为NULL，此时线程1通过if判断ptr不为空，将执行*ptr = ‘a’，这时候就发生错误，ptr暂时是一个野指针，而线程1试图给一个野指针赋值。类似这样的问题，可以通过加锁来避免，但具体的实现，也有很多需要注意的细节，防止产生死锁的错误。

- **内存泄漏与内存溢出**

内存泄漏是指程序申请了内存后(new)，用完的内存没有释放(delete)，一直被某个或某些实例所持有却不再被使用，导致该内存无法回收再分配。内存泄漏是造成内存溢出的主要原因之一，特别是对于服务器这类持久运行的程序，无论多少内存，迟早会被占用完，最终导致内存溢出。在编码过程中，要时刻谨记一次申请，一次回收，同时更要关注一些类库的对象是否需要显示回收。

#### (2) 程序卡死

- **死循环**

在在编程中，一个靠自身控制无法终止的程序称为“死循环”，一个最简单的死循环例子：

```
while(1) { 
    printf(*);
}
```

while的判断条件为1，则会一直重复打印”*”的操作。但实际编程中死循环并不会这么简单，还需要与功能模块的逻辑联系起来。

- **死锁**

死锁是进程死锁的简称，是指多个进程循环等待它方占有的资源而无限期地僵持下去的局面。线程1与线程2均需要A、B两个资源，但A、B资源都只能同时被一个线程使用，一个典型的死锁局面是这样的，线程1申请了A资源并将A资源加锁，等待B资源申请成功即可完成任务，释放资源，但此时线程2已经申请了B资源并加锁，等待A资源就绪，此时线程1与线程2互相等待对方资源，因此两个线程均无法推进，导致了死锁的局面。

从上面的例子也验证了产生死锁的四个必要条件，互斥条件、不可抢占条件、占有且申请条件以及循环等待条件，当且仅当四个条件同时满足时，才会产生死锁。因此，要想避免死锁可从破坏这四个条件入手，常见的有安全序列与银行家算法。当死锁发生时，可以通过使用工具来快速定位到死锁位置