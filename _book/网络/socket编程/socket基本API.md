# socket基本API

**1.socket函数**

```cpp
int socket(int domain,int type,int protocol)
```
参数：
1）domain:表示套接字使用的协议族，在Linux系统中支持多种协议族，对TCP/IP协议来说，选择AF_INET就足以
2）type:指定了套接字使用的服务类型，可能的类型有以下几种：
SOCK_STREAM:提供可靠的面向连接的Socker服务，多用于资料传输，如（TCP协议）
SOCK_DGRAM:是提供无保障的面向消息的Socket服务，主要用于网络上发广播信息，如UDP协议，提供无连接不可靠的数据报交付服务
SOCK_SEQPACKET:为固定最大长度的数据报提供有序的，可靠的，基于双向连接的数据传输路径
3）protocol:参数protocol指定了套接字使用的协议，常见的socket类型有IPPROTO_TCP,IPPTOTO_UDP，在IPv4中，只有TCP协议提供SOCK_STREAM这种可靠的服务，对于这两种协议，protocol的默认值均为0，因为当protocol的值为0时，会自动选择type类型对应的默认协议
返回值：
socket描述符，该值大于等于0，创建套接字失败的话会返回-1
**2.bind函数**
在套接口中，一个套接字只是用户程序与内核交互信息的按钮，它自身没有太多的信息，也没有网络协议地址和端口号等信息，在进行网络信息的时候，必须把一个套接字和一个IP地址或端口号相关联，这个过程就是捆绑的过程。
bind()函数用于将一个IP地址或者端口号与一个套接字进行绑定，许多时候内核会帮我们自动绑定一个IP地址与端口号，然后又是用户可能需要自己来完成这个绑定的过程，以满足实际应用的需要，最典型的情况是一个服务器进程需要绑定一个众所周知的地址和端口以等待客户来连接，作为服务端，这一步绑定的操作是必要的，作为客户端则不是必要的，因为内核会帮我们自动选择合适的IP地址和端口号

```cpp
int bind(int sockfd,const struct sockaddr *addr,socklen_t addrlen)
```
参数：
1）sockfd:即socket描述字，它是通过socket()函数创建了唯一一个socket。bind()函数就是给这个描述字绑定一个名字
2）addr:一个const struct sockaddr *指针指向要绑定给sockfd的协议地址，sockaddr_in
3）addrlen:对应的是地址的长度

**3.connect函数**
这个connect()函数用于客户端中，将sockfd与远端IP地址，端口号进行绑定，在TCP客户端中调用这个函数将发生握手过程(会发送一个TCP连接请求)，并最终建立一个TCP连接。而对于UDP协议来说，调用这个函数只是在sockfd中记录远端IP地址和端口号，而不发送任何数据，参数信息和bind()函数是一样的

```cpp
int connect(int sockfd,const struct sockaddr *addr,socklen_t addrlen)
```
参数：同bind;
返回值：函数调用成功则返回0，失败则返回-1
connect()函数是套接字连接操作，对于TCP协议来说，connect()函数操作成功之后代表对应的套接字已与远端主机建立了连接，可以发送与接收数据

**4.listen函数**
listen()函数只能在TCP服务器进程中使用，让服务器进程进入监听状态，等待客户端的连接请求，listen()一般在bind()函数之后调用，在accept()函数之前调用

```cpp
int listen(int s,int backlog)
```
参数：
1）s,由socket()函数返回的套接字描述符
2）backlog，描述sockfd的等待连接队列能够达到的最大值
返回值：函数调用成功返回0，失败返回-1

**5.accept函数**

```cpp
int accept(int s,struct sockaddr *addr,socklen_t * addrlen)
```
参数：
1）s,由socket()函数返回的套接字描述符；
2）addr，用来返回已连接的客户端IP地址和端口号
3）addrlen，用于返回addr所指向的地址结构体的字节长度。
返回值：
若连接成功，则返回一个socket描述符（非负），出错则为-1

为了能让TCP客户端能正常的连接到服务器，服务器必须遵守以下流程处理：
1.调用socket()函数创建对应的套接字类型
2.调用bind()函数将套接字绑定到本地的一个端口地址
3.调用listen()函数让服务器进程进入监听状态，等待客户端的连接请求
4.调用accept()函数处理到来的连接请求

如果accept()连接成功，那么返回值是由内核自动生成的一个全新描述符，代表与客户端的TCP连接，一个服务器仅仅创建一个监听套接字，它在服务器生命周期内一直存在，内核为每个由服务器进程接受的客户端连接创建一个已连接的套接字

**6.read,write函数**
至此服务器与客户端已经建立好连接了，可以调用网络IO进行读写操作了，实现了网络中不同进程间的通信，网络IO操作有下面几组
read()/write
recv()/send()
readv()/writev()
recvmsg()/sendmsg()
recvfrom()/sendto()

**7.close()函数**
客户端与服务器建立连接之后，会进行一些读写操作，完成读写操作就要关闭相应的socket描述字

```cpp
int clost(int fd)
```
close操作只是使相应的socket描述字的引用计数-1，只有当引用计数为0的时候，才能触发TCP客户端向服务器发送终止连接的请求


server.cpp代码
```cpp
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main(){
    //创建套接字
    int serv_sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

    //将套接字和IP、端口绑定
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));  //每个字节都用0填充
    serv_addr.sin_family = AF_INET;  //使用IPv4地址
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  //具体的IP地址
    serv_addr.sin_port = htons(1234);  //端口
    bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));

    //进入监听状态，等待用户发起请求
    listen(serv_sock, 20);

    //接收客户端请求
    struct sockaddr_in clnt_addr;
    socklen_t clnt_addr_size = sizeof(clnt_addr);
    int clnt_sock = accept(serv_sock, (struct sockaddr*)&clnt_addr, &clnt_addr_size);

    //向客户端发送数据
    char str[] = "Hello World!";
    write(clnt_sock, str, sizeof(str));
   
    //关闭套接字
    close(clnt_sock);
    close(serv_sock);

    return 0;
}


```

client.cpp代码

```cpp
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

int main(){
      int sock = socket(AF_INET,SOCK_STREAM,0);
      
      struct sockaddr_in serv_addr;
      memset(&serv_addr,0,sizeof(serv_addr));
      serv_addr.sin_family = AF_INET;
      serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
      serv_addr.sin_port = htons(1234);
      connect(sock,(struct sockaddr*)&serv_addr,sizeof(serv_addr));

      char buffer[40];
      read(sock,buffer,sizeof(buffer)-1);

      printf("Message from server: %s\n",buffer);

      close(sock);
      
      return 0;
}
```
