# select,poll,epoll

### I/O多路复用场景

* 当客户处理多个描述符时
* 一个客户端同时处理多个套接字
* 如果一个TCP服务器既要处理监听套接字，又要处理已连接套接
* 如果一个服务器既要处理TCP又要处理UDP
* 一个服务器要处理多个服务或者多个协议  
  

### select

```C++
#include<sys/select.h>
#include<sys/time.h>

int select(int maxfdp1,fd_set *readset,fd_set *writeset,fd_set *exceptset,const struct timeval *timeout) //返回值为就绪描述符的数目

FD_ZERO(int fd,fd_set* fds) //清空集合
FD_SET(int fd,fd_set* fds)  //将给定的描述符加入集合
FD_ISSET(int fd,fd_set* fds)  //判断指定描述符是否在集合中
FD_CLR(int fd,fd_set* fds)  //将给定的描述符从文件中删除

```

它仅仅知道了有I/O事件发生了，却不知道是哪几个流，我们只能无差别的轮询所有流，找出能读出数据，或者写入的流，对他们进行操作，所以select具有O(n)无差别轮询复杂度

select代码：

```C++
#include<stdio.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<stdlib.h>
#include<netinet/in.h>
#include<arpa/inet.h>
#include<unistd.h>
#include<error.h>
#include<assert.h>

#define _BACKLOG_ 5

int fds[64]; //文件描述符集合

//命令行参数出错
static void usage(const char *proc)
{
	printf("usage : %s [ip][port]\n",proc);
}

//服务器端TCP的链接绑定监听状态。
static int startup(char *ip,int port)
{
	assert(ip);
	
	int sock = socket(AF_INET,SOCK_STREAM,0);
	if(sock < 0)
	{
		perror("socket");
		exit(1);
	}

	struct sockaddr_in local;
	local.sin_family = AF_INET;
	local.sin_port = htons(port);
	local.sin_addr.s_addr = inet_addr(ip);

	int set = -1;
	if(setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &set, sizeof(set)) < 0)
	{   
	   	perror("setsoickopt");
		exit(4);
	}

	if(bind(sock,(struct sockaddr*)&local,sizeof(local)) < 0)
	{
		perror("bind");
		exit(2);
	}

	if(listen(sock,_BACKLOG_) < 0)
	{
		perror("listen");
		exit(3);
	}

	return sock;
}


int main(int argc,char *argv[])
{
	//判断命令参数行.
	if(argc != 3)
	{
		usage(argv[0]);
		exit(1);
	}

	int port = atoi(argv[2]);
	char *ip = argv[1];

	int listen_sock = startup(ip,port);
	int done = 0;
	int new_sock = -1;
	struct sockaddr_in client;
	socklen_t len = sizeof(client);
	
	int max_fd; //selet函数的第一个记录参数/
	
	//读写事件文件描述符
	fd_set reads;
	fd_set writes;

	int i = 0;
	int fds_num = sizeof(fds)/sizeof(fds[0]);
	
	//对文件描述符集合进行初始化
	for(;i < fds_num;++i)
	{
		fds[i] = -1;
	}

	//当前存在listen文件描述符。
	fds[0] = listen_sock;
	
	max_fd = fds[0];	//初次调用select函数设置max_fd的值；

	while(!done)
	{
		//初始化读写文件描述符
		FD_ZERO(&reads);
		FD_ZERO(&writes);
		//将listen_sock设置为reads，因为时在等待请求相应，相当与当前的读取操作。
		FD_SET(listen_sock,&reads);
		//设置多路复用中的轮寻时间表。
		struct timeval timeout = {5,0};
		
		//没次读取添加事件到reads中。;
		for(i = 1;i < fds_num; ++i)
		{
			if(fds[i] >0)
			{
				FD_SET(fds[i],&reads);
				if(fds[i] > max_fd)
				{
					max_fd = fds[i];
				}
			}
		}

		switch(select(max_fd+1, &reads,&writes,NULL,&timeout))
		{
			case 0 ://timeout
				{
					printf("select timeout\n");
					break;
				}

			case -1:
				{	//error	
					perror("select");
					break;
				}

			default:
				{	//返回改变了的文件描述.
					i = 0;
					char buf[1024];
					//遍历所有的文件描述符集合。
					for(;i<fds_num;++i)
					{
						//确认是否时监听时间,是的话就绪要accept；
						if(fds[i] == listen_sock && \
								FD_ISSET(fds[i],&reads))
						{
							new_sock = accept(listen_sock,(struct sockaddr*)&client,&len);
							if(new_sock <0)
							{
								perror("accept");
								continue;
							}
							printf("get a new connet...%d\n",new_sock);
							for(i = 0;i< fds_num;++i)
							{
								if(fds[i] == -1)
								{
									fds[i] = new_sock;
									break;
								}
							}
							if(i == fds_num)
							{
								close(new_sock);
							}
						}
		
						else if(fds[i] > 0 &&\
								FD_ISSET(fds[i],&reads))	//正常事件，但是是非监听时间，也就代表时新建立的new_sock。
								{
								//	char buf[1024];
									ssize_t s = read(fds[i],buf,sizeof(buf) -1);
									if(s > 0)
									{
										buf[s] = '\0';
									//	printf("client : %s\n",buf);
										printf("client : %s",buf);
										FD_SET(fds[i],&writes);
									//	write(fds[i],buf,sizeof(s)+1);
									}
									else if(s == 0)
									{
										printf("client quit...\n");
										close(fds[i]);
										fds[i] = -1;
									}
									else{}
								}
						else{}

						if(fds[i] > 0&&\
								FD_ISSET(fds[i],&writes))
						{
							write(fds[i],buf,sizeof(buf));
						}
					}
				}
		break;
		}
	}
	return 0;
}

```

**select缺点：**

* 单个进程所打开的FD是有限的，通过FD_SETSIZE设置，默认1024
* 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
* 对socket扫描时是线性扫描，采用轮询的方法，效率较低  
  

### poll

poll本质上和select没有区别，他将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态，但是它没有最大连接数的限制，因为它是基于链表存储的

```c++
int poll(struct pollfd *fdarray,unsigned long nfds,int timeout);
struct pollfd{
      int fd;           //需要监视的文件描述符
      short events;     //需要内核监视的时间
      short revents     //实际发生的事件 
}
```

poll只解决了select最大连接数限制的问题，但是对于其他select缺点还是存在  


### epoll

```C++
// 数据结构
// 每一个epoll对象都有一个独立的eventpoll结构体
// 用于存放通过epoll_ctl方法向epoll对象中添加进来的事件
// epoll_wait检查是否有事件发生时，只需要检查eventpoll对象中的rdlist双链表中是否有epitem元素即可
struct eventpoll {
    /*红黑树的根节点，这颗树中存储着所有添加到epoll中的需要监控的事件*/
    struct rb_root  rbr;
    /*双链表中则存放着将要通过epoll_wait返回给用户的满足条件的事件*/
    struct list_head rdlist;
};

// API
int epoll_create(int size); // 内核中间加一个 ep 对象，把所有需要监听的 socket 都放到 ep 对象中
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event); // epoll_ctl 负责把 socket 增加、删除到内核红黑树
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);// epoll_wait 负责检测可读队列，没有可读 socket 则阻塞进程
```

**epoll的优点：**

* 没有最大的并发连接的限制，能打开FD的上限远大于1024(1G内存上能监听大约10w的端口)
* 效率提升，不是轮询的方式，不会随着FD数目的增加效率下降，只有活跃可用的FD才会调用callback函数，即Epoll最大的优点就在于他只管你活跃的连接，而跟连接总数无关，因此在实际的网络环境中，Epoll的效率远高于select和poll
* 内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递，即epoll使用mmap减少复制开销
  **缺点：**
  只能工作在linux下  

**epoll LT与ET模式的区别**
LT时默认的模式，ET是“高速“模式
LT模式下，只要这个fd还有数据可读，每次epoll_wait都会返回它的事件，提醒用户程序去操作
ET模式下，他只会提示一次，直到下次再有数据流入之前都不会再提示了，无论fd中是否还有数据可读

* ET&&LT优缺点

|      | LT                                             | ET                                 |
| ---- | ---------------------------------------------- | ---------------------------------- |
| 优点 | 易于编码，未读完的数据下次还能继续读，不易遗漏 | 难以编码，需要一次读完，有时会遗漏 |
| 缺点 | 效率比较低                                     | 相对LT模式效率比较高               |

* LT比ET效率低的原因

  ET在通知用户后，就会把fd从就绪队列里删除。而LT通知用户后fd还在就绪链表中，随着fd的增多，就绪链表越大。下次epoll要通知用户时还需要遍历整个就绪链表。遍历的性能是线性，如果fd的数量非常多，就会带来比较显著的效率下降。同样数量的fd下，LT模式维护的就绪链表比ET的大。

### select poll epoll区别

|            | select                                           | poll                                             | epoll                                                        |
| ---------- | ------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| 操作方式   | 遍历                                             | 遍历                                             | 回调                                                         |
| 数据结构   | 数组                                             | 链表                                             | 红黑树                                                       |
| 最大连接数 | 1024                                             | 无上限                                           | 无上限                                                       |
| fd拷贝     | 每次调用select都需要把fd集合从用户态拷贝到内核态 | 每次调用select都需要把fd集合从用户态拷贝到内核态 | fd首次调用epoll_ctl拷贝，每次调用epoll_wait不拷贝            |
| 工作模式   | LT                                               | LT                                               | 支持ET高效模式                                               |
| 工作效率   | 每次调用都进行线性遍历，时间复杂度为O(n)         | 每次调用都进行线性遍历，时间复杂度为O(n)         | 事件通知方式，每当fd就绪，系统注册的回调函数就会被调用，将就绪fd放到readyList里面，时间复杂度O(1) |

  




### epoll一定比select高效么？

假设现在有1024个fd ； select 和epoll 都同时维护他， 假设这些fd 都是活跃的， 这种情况下，select一次扫描 可以扫描1024个fd，空闲的fd很少，

但是epoll 就有可能不一样了， epoll 是先注册等待回调， 有可能出现1024次回调；

这样的情况下， 要是说epoll 效率比select 高-----这就不好说了！！！！！！！！

如果select 和epoll 同时维护1024个fd ，但是每次只有一个fd有事件，这种情况下 select 每次都会扫描所有的fd， 对比于epoll 每次只有一个fd 回调。 select 做了很多无用功， 此时应该epoll的效率高吧！！

或者在短连接多的时候， 一个连接使用epoll 会触发epoll_ctrl_add/del 两次系统调用，但是select 只有一次扫描 ，此时 也许select 效率性能更高。

如果在并发量低，socket都比较活跃的情况下，select就不见得比epoll慢了
