---
title: 基于epoll的web服务器
date: 2024-04-12 16:37:58
tags: C++ 网络编程 WebServer
categories:
    网络编程
---

## 基于epoll的web服务器(C语言版本)

### 1. 初始化监听套接字

包括创建监听套接字，设置端口复用，绑定，设置监听等步骤

#### 1.1 创建监听套接字（socket函数）

`socket()`打开一个网络通讯端口，如果成功的话，就像`open()`一样返回一个文件描述符，应用程序可以像读写文件一样用`read/write`在网络上收发数据，如果`socket()`调用出错则返回`-1`。对于`IPv4`，`domain`参数指定为`AF_INET`。对于`TCP`协议，`type`参数指定为`SOCK_STREAM`，表示面向流的传输协议。如果是`UDP`协议，则`type`参数指定为`SOCK_DGRAM`，表示面向数据报的传输协议。`protocol`参数的介绍从略，指定为0即可。

```txt
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
domain:
	AF_INET 这是大多数用来产生socket的协议，使用TCP或UDP来传输，用IPv4的地址
	AF_INET6 与上面类似，不过是来用IPv6的地址
	AF_UNIX 本地协议，使用在Unix和Linux系统上，一般都是当客户端和服务器在同一台及其上的时候使用
type:
	SOCK_STREAM 这个协议是按照顺序的、可靠的、数据完整的基于字节流的连接。这是一个使用最多的socket类型，这个socket是使用TCP来进行传输。
	SOCK_DGRAM 这个协议是无连接的、固定长度的传输调用。该协议是不可靠的，使用UDP来进行它的连接。
	SOCK_SEQPACKET该协议是双线路的、可靠的连接，发送固定长度的数据包进行传输。必须把这个包完整的接受才能进行读取。
	SOCK_RAW socket类型提供单一的网络访问，这个socket类型使用ICMP公共协议。（ping、traceroute使用该协议）
	SOCK_RDM 这个类型是很少使用的，在大部分的操作系统上没有实现，它是提供给数据链路层使用，不保证数据包的顺序
protocol:
	传0 表示使用默认协议。
返回值：
	成功：返回指向新创建的socket的文件描述符，失败：返回-1，设置errno
```



#### 1.2 设置端口复用（setsockopt函数）

在`server`的TCP连接没有完全断开之前不允许重新监听是不合理的。因为，TCP连接没有完全断开指的是`connfd`（127.0.0.1:6666）没有完全断开，而我们重新监听的是`listenfd`（0.0.0.0:6666），虽然是占用同一个端口，但`IP`地址不同，`connfd`对应的是与某个客户端通讯的一个具体的`IP`地址，而`listenfd`对应的是`wildcard address`。解决这个问题的方法是使用`setsockopt()`设置`socket`描述符的选项`SO_REUSEADDR`为1，表示允许创建端口号相同但`IP`地址不同的多个`socket`描述符。

在server代码的socket()和bind()调用之间插入如下代码：

```c
int opt = 1;
setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
```

#### 1.3 绑定（bind函数）

服务器程序所监听的网络地址和端口号通常是固定不变的，客户端程序得知服务器程序的地址和端口号后就可以向服务器发起连接，因此服务器需要调用`bind`绑定一个固定的网络地址和端口号。

```txt
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
sockfd：
	socket文件描述符
addr:
	构造出IP地址加端口号
addrlen:
	sizeof(addr)长度
返回值：
	成功返回0，失败返回-1, 设置errno
```

`bind()`的作用是将参数`sockfd`和`addr`绑定在一起，使`sockfd`这个用于网络通讯的文件描述符监听`addr`所描述的地址和端口号。前面讲过，`struct sockaddr *`是一个通用指针类型，`addr`参数实际上可以接受多种协议的`sockaddr`结构体，而它们的长度各不相同，所以需要第三个参数`addrlen`指定结构体的长度。如：

```c
struct sockaddr_in addr;
bzero(&addr, sizeof(addr));
servaddr.sin_family = AF_INET;
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);	//INADDR_ANY = 0
servaddr.sin_port = htons(8888);
```

首先将整个结构体清零，然后设置地址类型为`AF_INET`，**网络地址为**`INADDR_ANY`**，这个宏表示本地的任意`IP`**地址**，因为服务器可能有多个网卡，每个网卡也可能绑定多个`IP`地址，这样设置可以在所有的`IP`地址上监听，直到与某个客户端建立了连接时才确定下来到底用哪个`IP`地址，端口号为8888。

#### 1.4 设置监听 （listen函数）

```txt
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
int listen(int sockfd, int backlog);
sockfd:
	socket文件描述符
backlog:
	排队建立3次握手队列和刚刚建立3次握手队列的链接数和(现在只表示建立链接队列的数量)
```

查看系统默认`backlog`

```shell
cat /proc/sys/net/ipv4/tcp_max_syn_backlog
```

典型的服务器程序可以同时服务于多个客户端，当有客户端发起连接时，服务器调用的`accept()`返回并接受这个连接，如果有大量的客户端发起连接而服务器来不及处理，尚未`accept`的客户端就处于连接等待状态，`listen()`声明`sockfd`处于监听状态，并且最多允许有`backlog`个客户端处于连接待状态，如果接收到更多的连接请求就忽略。`listen()`成功返回0，失败返回-1。

#### 1.5 初始化监听套接字（initListenFd函数）

```c
// 初始化监听套接字
int initListenFd(port){
    // 1. 创建监听套接字
    int lfd = socket(AF_INET,SOCK_STREAM,0);
    if(lfd == -1){
        perror("socket error");
        return -1;
    }
    // 2. 设置端口复用
    int opt = 1;
    int ret = setsockopt(lfd,SOL_SOCKET,SO_REUSEADDR,&opt,sizeof(opt));
    if(ret == -1){
        perror("setsockopt error");
        return -1;
    }
    // 3. 绑定
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = 0;
    int ret = bind(lfd,(struct sockaddr *)&addr,sizeof(addr));
    if(ret == -1){
        perror("bind error");
        return -1;
    }
    // 4.设置监听
    ret = listen(lfd,128);
    if(ret == -1){
        perror("listen error");
        return -1;
    }
    // 5. 返回fd
    return lfd;
}
```



### 2. 启动epoll

`epoll`是`Linux`下**IO多路复用**接口`select/poll`的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率，因为它会复用文件描述符集合来传递结果而不用迫使开发者每次等待事件之前都必须重新准备要被侦听的文件描述符集合，另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入`Ready`队列的描述符集合就行了。

`epoll`除了提供`select/poll`那种IO事件的水平触发（Level Triggered）外，还提供了边沿触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少`epoll_wait/epoll_pwait`的调用，提高应用程序效率。

#### 2.1 创建epoll树  (epoll_create)

 创建一个`epoll`句柄，参数`size`用来告诉内核监听的文件描述符的个数，跟内存大小有关。(**参数size已经弃用，只需提供大于0的数字就行**)

```txt
#include <sys/epoll.h>
int epoll_create(int size)		
size：监听数目（内核参考值）
返回值：成功：非负文件描述符；失败：-1，设置相应的errno
```

可以使用cat命令查看一个进程可以打开的socket描述符上限。

```shell
cat /proc/sys/fs/file-max
806425
```

如有需要，可以通过修改配置文件的方式修改该上限值。

```txt
sudo vi /etc/security/limits.conf
在文件尾部写入以下配置,soft软限制，hard硬限制。如下图所示。
* soft nofile 65536
* hard nofile 100000
```

![image-20231026200903197](https://glf-1309623969.cos.ap-shanghai.myqcloud.com/img/image-20231026200903197.png)

#### 2.2 上树（epoll_ctl函数）

控制某个`epoll`监控的文件描述符上的事件：注册、修改、删除。

```txt
#include <sys/epoll.h>
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)
    epfd：	为epoll_creat的句柄
    op：		表示动作，用3个宏来表示：
    EPOLL_CTL_ADD (注册新的fd到epfd)，
    EPOLL_CTL_MOD (修改已经注册的fd的监听事件)，
    EPOLL_CTL_DEL (从epfd删除一个fd)；
    event：	告诉内核需要监听的事件

struct epoll_event {
    __uint32_t events; /* Epoll events */
    epoll_data_t data; /* User data variable */
};

typedef union epoll_data {
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;

EPOLLIN ：	表示对应的文件描述符可以读（包括对端SOCKET正常关闭）
EPOLLOUT：	表示对应的文件描述符可以写
EPOLLPRI：	表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）
EPOLLERR：	表示对应的文件描述符发生错误
EPOLLHUP：	表示对应的文件描述符被挂断；
EPOLLET： 	将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)而言的
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
返回值：成功：0；失败：-1，设置相应的errno
```

#### 2.3 检测（epoll_wait函数）

等待所监控文件描述符上有事件的产生，类似于`select()`调用。

```txt
#include <sys/epoll.h>
	int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)
    events：		用来存内核得到事件的集合，可简单看作数组。
    maxevents：	告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，
    timeout：	是超时时间
    -1：	阻塞
    0：	立即返回，非阻塞
    >0：	指定毫秒
    返回值：	成功返回有多少文件描述符就绪，时间到时返回0，出错返回-1
```

#### 2.4 启动epoll(epollrun函数)

```c
//启动epoll
void epollrun(int lfd){
    // 1. 创建epoll树
    int epfd = epoll_create(1);
    if(epfd == -1){
        perror("epoll_create error");
        return -1;
    }
    // 2. lfd上树
    struct epoll_event ev;
    ev.data.fd = lfd;
    ev.events = EPOLLIN;
    int ret = epoll_ctl(epfd,EPOLL_CTL_ADD,lfd,&ev);
    if(ret == -1){
        perror("epoll_ctl error");
        return -1;
    }
    // 3. 检测(委托内核检测添加到树上的节点)
    struct epoll_event evs[1024];
    int size = siezof(evs) / sizeof(struct epoll_event);
    while(1){
        int num = epoll_wait(epfd,evs,size,-1);
        if(num == -1) {
            perror("epoll_wait error");
            return -1;
        }
        // 遍历发生变化的节点
        for(int i = 0; i < num; ++i){
            if(!(evs[i].events & EPOLLIN)) {
                // 不是读事件
                continue;
            }
            int fd = evs[i].data.fd;
            if(fd == lfd){
                // 建立新连接 accept
                acceptClient(lfd,epfd);
            }else{
                // 主要是接受对端的数据(读数据)
                recvHttpRequest(fd,epfd);
            }
        }
    }

}
```

### 3. 建立连接

#### 3.1 建立连接 （accept函数）

**三方握手完成后，服务器调用`accept()`接受连接**，如果服务器调用`accept()`时还没有客户端的连接请求，就阻塞等待直到有客户端连接上来。`addr`是一个传出参数，`accept()`返回时传出客户端的地址和端口号。`addrlen`参数是一个传入传出参数（value-result argument），传入的是调用者提供的缓冲区`addr`的长度以避免缓冲区溢出问题，传出的是客户端地址结构体的实际长度（有可能没有占满调用者提供的缓冲区）。如果给`addr`参数传`NULL`，表示不关心客户端的地址。

```txt
#include <sys/types.h> 		/* See NOTES */
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
sockdf:
	socket文件描述符
addr:
	传出参数，返回链接客户端地址信息，含IP地址和端口号
addrlen:
	传入传出参数（值-结果）,传入sizeof(addr)大小，函数返回时返回真正接收到地址结构体的大小
返回值：
	成功返回一个新的socket文件描述符，用于和客户端通信，失败返回-1，设置errno

```

我们的服务器程序结构是这样的：

```c
while (1) {
	cliaddr_len = sizeof(cliaddr);
	connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);
	n = read(connfd, buf, MAXLINE);
	......
	close(connfd);
}
```

整个是一个while死循环，每次循环处理一个客户端连接。由于`cliaddr_len`是传入传出参数，每次调用`accept()`之前应该重新赋初值。`accept()`的参数`listenfd`是先前的监听文件描述符，而`accept()`的返回值是另外一个文件描述符`connfd`，之后与客户端之间就通过这个`connfd`通讯，最后关闭`connfd`断开连接，而不关闭`listenfd`，再次回到循环开头`listenfd`仍然用作`accept`的参数。`accept()`成功返回一个文件描述符，出错返回-1。

#### 3.2 epoll事件模型

`EPOLL`事件有两种模型：

- `Edge Triggered (ET) `边缘触发只有数据到来才触发，不管缓存区中是否还有数据。

- `Level Triggered (LT) `水平触发只要有数据都会触发。

```txt
思考如下步骤：
1.	假定我们已经把一个用来从管道中读取数据的文件描述符(rfd)添加到epoll描述符。
2.	管道的另一端写入了2KB的数据
3.	调用epoll_wait，并且它会返回rfd，说明它已经准备好读取操作
4.	读取1KB的数据
5.	调用epoll_wait……
```

**ET模式 即Edge Triggered工作模式（边沿触发）**

如果我们在第1步将`rfd`添加到`epoll`描述符的时候使用了`EPOLLET`标志，那么在第5步调用`epoll_wait`之后将有可能会挂起，因为剩余的数据还存在于文件的输入缓冲区内，而且数据发出端还在等待一个针对已经发出数据的反馈信息。只有在监视的文件句柄上发生了某个事件的时候 `ET` 工作模式才会汇报事件。因此在第5步的时候，调用者可能会放弃等待仍在存在于文件输入缓冲区内的剩余数据。`epoll`工作在`ET`模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。最好以下面的方式调用`ET`模式的`epoll`接口，在后面会介绍避免可能的缺陷。

-   基于非阻塞文件句柄

-   只有当`read`或者`write`返回`EAGAIN`(非阻塞读，暂时无数据)时才需要挂起、等待。但这并不是说每次`read`时都需要循环读，直到读到产生一个`EAGAIN`才认为此次事件处理完成，当`read`返回的读到的数据长度小于请求的数据长度时，就可以确定此时缓冲中已没有数据了，也就可以认为此事读事件已处理完成。

**LT模式即Level Triggered工作模式(水平触发)**

与`ET`模式不同的是，以`LT`方式调用`epoll`接口的时候，它就相当于一个速度比较快的`poll`，无论后面的数据是否被使用。

**比较**

`LT(level triggered)`：`LT`是**缺省**的工作方式，并且同时支持`block`和`no-block socket`。在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的`fd`进行IO操作。如果你不作任何操作，内核还是会继续通知你的，所以，这种模式编程出错误可能性要小一点。**传统的`select/poll`**都是这种模型的代表。

`ET(edge-triggered)`：**`ET`是高速工作方式，只支持`no-block socket`**。在这种模式下，当描述符从未就绪变为就绪时，内核通过`epoll`告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知。请注意，如果一直不对这个`fd`作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知`**(only once)**`.

#### 3.3 阻塞与非阻塞

- 非阻塞模式可以理解为，执行此套接字的网络调用时，不管是否执行成功，都会立即返回。


​		如调用`recv( )`函数读取网络缓冲区中的数据时，不管是否读到数据都立即返回，而不会一直挂在此函数的调用上。

- 阻塞模式为只有接收到数据后才会返回，套接字默认的会创建堵塞模式。

#### 3.4 建立连接（accpetClient函数）

```c
int accpetClient(int lfd,int epfd){
    // 1. 建立连接
    struct sockaddr_in cliaddr;
    socklen_t len = sizeof(cliaddr);
    cliaddr.sin_family = AF_INET;
    int cfd = accept(lfd,(struct sockaddr*)&cliaddr,&len);
    if(cfd == -1){
        perror("accept error");
        return -1;
    }
    char ip[16]="";
    printf("new client ip=%s port=%d\n",
    inet_ntop(AF_INET, &cliaddr.sin_addr.s_addr,ip,16),ntohs(cliaddr.sin_port));
    
    // 2. 设置非阻塞
    int flag = fcntl(cfd,F_GETFL);
    flag |= O_NONBLOCK;
    fcntl(cfd,F_SETFL,flag);

    // 3. cfd添加到epoll
    struct epoll_event ev;
    ev.data.fd = cfd;
    ev.events = EPOLLIN | EPOLLET;      //边沿模式

    int ret = epoll_ctl(epfd,EPOLL_CTL_ADD,cfd,&ev);
    if(ret == -1){
        perror("epoll_ctl error");
        return -1;
    }

    return 0;
}
```



### 4. 接收客户端发来的http请求

#### 4.1 接收数据 （recv函数）

接收来自`socket`缓冲区的数据，当缓冲区没有数据可取时，`recv`会一直处于阻塞状态()，直到缓冲区至少又一个字节数据可读取，或者对端关闭，并读取所有数据后返回。

```txt
#include<sys/types.h>
#include<sys/socket.h>

int recv(int sockfd, char * buf, int len, int flags);
sockfd：连接的fd
buf：用于接收数据的缓冲区
len：缓冲区长度
flags：指定调用方式
返回值：成功返回实际读到的字节数。如果recv在copy时出错，那么它返回err，err小于0；如果recv函数在等待协议接收数据时网络中断了，那么它返回0。
```

**read**

`read`函数从文件描述符（包括`TCP Socket`）中读取数据，并将读取的数据存储到指定的缓冲区中。

```txt
ssize_t read(int fd, void *buf, size_t count);
fd：要读取数据的文件描述符，可以是TCP Socket。
buf：存储读取数据的缓冲区。
count：要读取的字节数。
返回值：成功时返回实际读取的字节数，失败时返回-1，并设置errno变量来指示错误的原因。
```

`read`函数和`recv`函数都是阻塞调用，即在没有数据可读时会一直阻塞等待。它们的主要区别在于`recv`函数可以通过`flags`参数控制一些特殊的行为，如设置`MSG_PEEK`标志来预览数据而不将其从缓冲区中移除。

#### 4.2 EAGAIN错误

以`O_NONBLOCK`的标志打开文件`/socket/FIFO`，如果你连续做`read`或者`recv`操作而没有数据可读。此时程序不会阻塞起来等待数据准备就绪返回，`read`函数会返回一个错误`EAGAIN`，提示你的应用程序现在没有数据可读请稍后再试。

```txt
（epoll的ET模式下设置recv，对应的fd文件描述符设置为非阻塞）下调用了阻塞操作，在该操作没有完成就返回这个错误，这个错误不会破坏socket的同步，不用管它，下次循环接着recv就可以。对非阻塞socket而言，EAGAIN不是一种错误。在VxWorks和Windows上，EAGAIN的名字叫做EWOULDBLOCK。
```

#### 4.3 接受http请求（recvHttpRequest函数）

```c
int recvHttpRequest(int cfd,int epfd){
    char buf[4096] = { 0 };
    char tmp[1024] = { 0 };
    int len = 0;
    int total = 0;
    // 1. 接收数据
    while((len = recv(cfd,tmp,sizeof(tmp),0)) > 0){
        if(total + len < sizeof(buf)){
            memcpy(buf + total,tmp,len);
        }
        total += len;
    }

    // 2. 判断数据是否接受完毕
    if(len == -1 && errno == EAGAIN){
        // 解析请求行   
        /*
        0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
        G E T   / 1 . t x t H  T  T  P  /  1  .  1  /r /n
        */
        char* pt = strstr(buf,"\r\n");  //大字符串找小字符串
        int reqLen = pt - buf;
        buf[reqLen] = '\0';

    }
    else if(len == 0){
        // 客户端断开连接
        epoll_ctl(epfd,EPOLL_CTL_DEL,cfd,NULL);
    }
    else{
        perror("recv error");
    }

    return 0;
}
```

### 5. 解析请求行

#### 5.1 格式化拆分字符串 （sscanf函数）

```txt
sprintf（）是把格式化数据输出成（存储到）字符串。
sscanf（）是从字符串中读取格式化的数据。
```

```txt
// 函数原型
// 将参数str的字符串根据参数format字符串来转换并格式化数据，转换后的结果存于对应的参数内。
sscanf(const char *str, const char *format, ...)。

具体功能如下：
（1）根据格式从字符串中提取数据。如从字符串中取出整数、浮点数和字符串等。
（2）取指定长度的字符串
（3）取到指定字符为止的字符串
（4）取仅包含指定字符集的字符串
（5）取到指定字符集为止的字符串

// 可以使用正则表达式进行字符串的拆分
// shell脚本的时候, 会将正则表达式, 其实就是字符串的匹配规则, 用特殊字符来描述一类字符串
/*
正则匹配规则:
	[1-9]: 匹配一个字符, 这个字符在 1-9 范围内就满足条件
	[2-7]: 匹配一个字符, 这个字符在 2-7 范围内就满足条件
	[a-z]: 匹配一个字符, 这个字符在 a-z 范围内就满足条件
	[A,b,c,D, e, f]: 匹配一个字符, 这个字符是集合中任意一个就满足条件
	[1-9, f-x]: 匹配一个字符, 这个字符是1-9, 或者f-x 集合中的任意一个就满足条件
	[^1]: ^代表否定, 匹配一个字符,这个字符只要不是1就满足条件
	[^2-8]: 匹配一个字符,这个字符只要不在 2-8 范围内就满足条件
	[^a-f]: 匹配一个字符,这个字符只要不在 a-f 范围内就满足条件
	[^ ]: 匹配一个字符,这个字符只要不是空格就满足条件
使用正则表达式如何取匹配字符串:
举例: 
	字符串 ==> abcdefg12345AABBCCDD890
	正则表达式: [1-9][a-z], 可以匹配两个字符
	匹配方式: 从原始字符串开始位置遍历, 每遍历一个字符都需要和正则表达式进行匹配, 
		满足条件继续向后匹配, 不满足条件, 匹配结束
		从新开始: 从正则表达式的第一个字符重新开始向后一次匹配
			当整个大字符串被匹配一遍, 就结束了
	abcdefg12345AABBCCDD893b
		- 匹配到一个子字符串: 3b
	1a2b3c4d5e6f7g12345AABBCCDD893b
	 - 1a
	 - 2b
	 - 3c
	 - 4d
	 - 5e
	 - 6f
	 - 7g
	 - 3b
*/
sscanf可以支持格式字符%[]：

(1)-: 表示范围，如：%[1-9]表示只读取1-9这几个数字 %[a-z]表示只读取a-z小写字母，类似地 %[A-Z]只读取大写字母
(2)^: 表示不取，如：%[^1]表示读取除'1'以外的所有字符 %[^/]表示除/以外的所有字符
(3),: 范围可以用","相连接 如%[1-9,a-z]表示同时取1-9数字和a-z小写字母 
(4)原则：从第一个在指定范围内的数字开始读取，到第一个不在范围内的数字结束%s 可以看成%[] 的一个特例 %[^ ](注意^后面有一个空格！)
```

#### 5.2 转码

```txt
假设浏览器访问的文件名中有中文: Linux内核.jpg
	- 浏览器在给服务器发送请求的时候, 会自动将中文进制转换: Linux%E5%86%85%E6%A0%B8.jpg
	- 为什么要转换?
		- 在http请求的请求行中不支持中文字符, 如果有中文, 浏览器就会自动将中文进行转换
		- 在服务器端收到的文件名就不是原来的名字了, 因此服务器端就不能识别了
		- 如果服务器端想要正确的处理, 需要将特殊字符串解析成原来的汉字
		
$ unicode 内
UTF-8: e5 86 85 
$ unicode 核
UTF-8: e6 a0 b8
```

#### 5.3 获取文件信息（stat）

`Linux` 下可以使用`stat `命令查看文件的属性，其实这个命令内部就是通过调用` stat()`函数来获取文件属性的，`stat `函数是 `Linux `中的系统调用，用于获取文件相关的信息。

```txt
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
int stat(const char *pathname, struct stat *buf);
```

```txt
struct stat
{
	 dev_t st_dev; /* 文件所在设备的 ID */
	 ino_t st_ino; /* 文件对应 inode 节点编号 */
	 mode_t st_mode; /* 文件对应的模式 */
	 nlink_t st_nlink; /* 文件的链接数 */
	 uid_t st_uid; /* 文件所有者的用户 ID */
	 gid_t st_gid; /* 文件所有者的组 ID */
	 dev_t st_rdev; /* 设备号（指针对设备文件） */
	 off_t st_size; /* 文件大小（以字节为单位） */
	 blksize_t st_blksize; /* 文件内容存储的块大小 */
	 blkcnt_t st_blocks; /* 文件内容所占块数 */
	 struct timespec st_atim; /* 文件最后被访问的时间 */
	 struct timespec st_mtim; /* 文件内容最后被修改的时间 */
	 struct timespec st_ctim; /* 文件状态最后被改变的时间 */
};
st_dev：该字段用于描述此文件所在的设备。不常用，可以不用理会。
st_ino：文件的 inode 编号。
st_mode：该字段用于描述文件的模式，譬如文件类型、文件权限都记录在该变量中。
st_nlink：该字段用于记录文件的硬链接数，也就是为该文件创建了多少个硬链接文件。链接文件可以分为软链接（符号链接）文件和硬链接文件。
st_uid、st_gid：此两个字段分别用于描述文件所有者的用户 ID 以及文件所有者的组 ID。
st_rdev：该字段记录了设备号，设备号只针对于设备文件，包括字符设备文件和块设备文件，不用理会。
st_size：该字段记录了文件的大小（逻辑大小），以字节为单位。
st_atim、st_mtim、st_ctim：此三个字段分别用于记录文件最后被访问的时间、文件内容最后被修改的时间以及文件状态最后被改变的时间，都是 struct timespec 类型变量。
```

#### 5.3 解析请求行（parseRequestLine函数）

```c
int parseRequestLine(const char* line,int cfd){
    // 1. 拆分http请求行   get /xxx/1.jpg http/1.1
    char method[12];    // 方法
    char path[1024];    // 路径
    char protocol[12];  // 协议
    sscanf(line,"%[^ ] %[^ ] %[^ ]",method,path,protocol);
    printf("method = %s, path = %s, protocol = %s\n", method, path, protocol);

    // 判断是否是get请求
    if(strcasecmp(method,"get") != 0){     //不区分大小写
        return -1;
    }

    // 转码 将不能识别的中文乱码 -> 中文
    // 解码 %23 %34 %5f
    decode_str(path, path);

    // 2. 处理客户端请求的静态资源
    char* file = NULL;
    // 如果没有指定访问的资源, 默认显示资源目录中的内容
    if(strcmp(path,"/") == 0){
        // file的值, 资源目录的当前位置
        file = "./";
    }else{
        // 去掉path中的/ 获取访问文件名
        file = path + 1;
    }
    
    // 3. 获取文件属性
    struct stat st;
    int ret = stat(file,&st);
    if(ret == -1){
        // 文件不存在--回复404

        return 0;
    }
    // 判断文件类型（判断是目录还是文件）
    if(S_ISDIR(st.st_mode)){    // 目录
        // 把目录发给客户端
    }else{
        // 把文件内容发给客户端
    }
    return 0;
}
```

### 6. 发送响应头

```c
int sendHeadMsg(int cfd,int status,const char* desrc,const char* type,int length){
    // 状态行
    char buf[4096] = { 0 };
    sprintf(buf,"http/1.1 %d %s \r\n",status,desrc);
    // 消息报头
    sprintf(buf + strlen(buf),"Content-Type: %s\r\n",type);
    sprintf(buf + strlen(buf),"Content-Length: %d\r\n",length);

    send(cfd,buf,strlen(buf),0);
     // 空行
    send(cfd, "\r\n", 2, 0);
    return 0;
}
```

### 7. 通过文件名获取文件的类型

```c
// 通过文件名获取文件的类型
const char *get_file_type(const char *name)
{
    char* dot;

    // 自右向左查找‘.’字符, 如不存在返回NULL
    dot = strrchr(name, '.');   
    if (dot == NULL)
        return "text/plain; charset=utf-8";
    if (strcmp(dot, ".html") == 0 || strcmp(dot, ".htm") == 0)
        return "text/html; charset=utf-8";
    if (strcmp(dot, ".jpg") == 0 || strcmp(dot, ".jpeg") == 0)
        return "image/jpeg";
    if (strcmp(dot, ".gif") == 0)
        return "image/gif";
    if (strcmp(dot, ".png") == 0)
        return "image/png";
    if (strcmp(dot, ".css") == 0)
        return "text/css";
    if (strcmp(dot, ".au") == 0)
        return "audio/basic";
    if (strcmp( dot, ".wav" ) == 0)
        return "audio/wav";
    if (strcmp(dot, ".avi") == 0)
        return "video/x-msvideo";
    if (strcmp(dot, ".mov") == 0 || strcmp(dot, ".qt") == 0)
        return "video/quicktime";
    if (strcmp(dot, ".mpeg") == 0 || strcmp(dot, ".mpe") == 0)
        return "video/mpeg";
    if (strcmp(dot, ".vrml") == 0 || strcmp(dot, ".wrl") == 0)
        return "model/vrml";
    if (strcmp(dot, ".midi") == 0 || strcmp(dot, ".mid") == 0)
        return "audio/midi";
    if (strcmp(dot, ".mp3") == 0)
        return "audio/mpeg";
    if (strcmp(dot, ".ogg") == 0)
        return "application/ogg";
    if (strcmp(dot, ".pac") == 0)
        return "application/x-ns-proxy-autoconfig";

    return "text/plain; charset=utf-8";
}
```

### 8. 发送文件

#### 8.1 断言（assert函数）

编译期`assert`函数的目的在于当条件不满足时，阻止编译，从而防止错误的逻辑通过编辑。而运行期`assert`的目的在于运行时发现条件不满足时，产生一个`Debug`事件(`DebugBreak`)，从而让调试器停下来方便用户检查原因。`assert `是一个宏，不是函数。

```
//表达式可以是任何有效的 C 语言表达式，很多时候它是一个条件。
void assert(int expression or variable);
```

#### 8.2 光标函数（lseek函数）

```txt
#include <sys/types.h> 
#include <unistd.h>
off_t lseek(int handle, off_t offset, int fromwhere);
1) 欲将读写位置移到文件开头时:
lseek（int fildes,0,SEEK_SET）；
2) 欲将读写位置移到文件尾时:
lseek（int fildes，0,SEEK_END）；
3) 想要取得目前文件位置时:
lseek（int fildes，0,SEEK_CUR）；
```

#### 8.3 发送文件（sendFile函数）

```c
int sendFile(const char* filename,int cfd){

    // 1. 打开文件
    int fd = open(filename,O_RDONLY);
    assert(fd > 0);     // 断言
    // if(fd == -1){
    //     perror("open error");
    // }
    // 2. 循环读文件
#if 1
    char buf[4096] = { 0 };
    int len = 0, ret = 0;
    while((len = read(fd,buf,sizeof(buf))) > 0){
        // 发送读出的数据
        ret = send(cfd,buf,len,0);
        if(ret == -1){
            if(errno = EAGAIN){
                perror("send error:");
                continue;
            }else if (errno == EINTR) {
                perror("send error:");
                continue;
            } else {
                perror("send error:");
                return -1;
            }
        }
    }
#else
    off_t offset = 0;
    int size = lseek(fd,0,SEEK_END);
    lseek(fd,0,SEEK_SET);
    while(offset < size){
        int ret = sendfile(cfd,fd,&offset,size);
        printf("ret value: %d\n",ret);
        if(ret == -1 && errno == EAGAIN){
            printf("没数据。。。\n");
            perror("snedfile");
            
        }
    }
#endif
    close(fd);
    return 0;
}
```

### 9. 发送目录

#### 9.1 目录扫描函数（scandir函数）

`scandir()`会扫描参数`dir`指定的目录文件，经由参数`select`指定的函数来挑选目录结构至参数`namelist`数组中，最后再调用参数`compar`指定的函数来排序`namelist`数组中的目录数据。每次从目录文件中读取一个目录结构后便将此结构传给参数`select`所指的函数，`select`函数若不想要将此目录结构复制到`namelis`t数组就返回0，若`select`为空指针则代表选择所有的目录结构。`scandir()`会调用`qsort()`来排序数据，参数`compar`则为`qsort()`的参数，若是要排列目录名称字母则可使用`alphasort()`。

```txt
#include <dirent.h>
int scandir(const char *dir, 
			struct dirent ***namelist,
			int (*select)(const struct dirent *),
			int (*compar)(const struct dirent **, 
			const struct dirent **));
dir:指定扫描的目录
namelist:struct dirent结构体类型的三级指针，用于获取该函数内部为存放返回结果的分配的动态内存
select:函数指针，指向过滤模式函数,当selectr指针设置为NULL时，扫描dir目录下的所有顶层文件.该函数有一个参数const struct dirent *是指在遍历过程中所遍历到的每一个子目录dirent，select可以根据dirent的类型、名称等信息来判定当前的dirent是否为合法的子目录，合法则函数返回0，则该子目录的名称会被存储在namelist中；否则返回非0，则该子目录被过滤掉。
compar:函数指针，指向对遍历结果进行排序函数，alphasort函数和versionsort是经常用到的函数
```

#### 9.2 发送目录（sendDir函数）

```c
// 发送目录内容
int sendDir(const char* dirname, int cfd)
{
   
   // 拼接一个html页面<table></table>
   char buf[4096] = { 0 };
   
   sprintf(buf,"<html><head><title>目录名：%s</title></head><body><table>",dirname);
   //sprintf(buf + strlen(buf),"<body><h1>当前目录：%s</h1><table>",dirname);

    // 目录项二级指针
    struct dirent** ptr;
    int num = scandir(dirname,&ptr,NULL,alphasort);

    // 遍历目录
    for(int i = 0; i < num; i++){
        // 取出文件名 namelist 指向的是一个指针数组 struct dirent* tmp[]
        char* name = ptr[i]->d_name;
        char subPath[1024] = { 0 };
        // 拼接文件袋完整路径
        sprintf(subPath,"%s/%s",dirname,name);
        
        struct stat st;
        stat(subPath,&st);

        char enstr[1024] = {0};
        // 编码生成 %E5 %A7 之类的东西
        encode_str(enstr, sizeof(enstr), name);

        // 如果是文件
        if(S_ISREG(st.st_mode)) {       
            sprintf(buf+strlen(buf), 
                    "<tr><td><a href=\"%s\">%s</a></td><td>%ld</td></tr>",
                    enstr, name, (long)st.st_size);
        } else if(S_ISDIR(st.st_mode)) {		// 如果是目录       
            sprintf(buf+strlen(buf), 
                    "<tr><td><a href=\"%s/\">%s/</a></td><td>%ld</td></tr>",
                    enstr, name, (long)st.st_size);
        }
        int ret = send(cfd, buf, strlen(buf), 0);
        if (ret == -1) {
            if (errno == EAGAIN) {
                perror("send error:");
                continue;
            } else if (errno == EINTR) {
                perror("send error:");
                continue;
            } else {
                perror("send error:");
                return -1;
            }
        }
        memset(buf, 0, sizeof(buf));
        // 字符串拼接
        free(ptr[i]);
    }  
    
    // 字符串拼接
    //memset(buf, 0, sizeof(buf));
    sprintf(buf, "</table></body></html>");
    send(cfd, buf, strlen(buf), 0);
    printf("dir message send OK!!!!\n"); 
#if 0
    // 打开目录
    DIR* dir = opendir(dirname);
    if(dir == NULL)
    {
        perror("opendir error");
        exit(1);
    }

    // 读目录
    struct dirent* ptr = NULL;
    while( (ptr = readdir(dir)) != NULL )
    {
        char* name = ptr->d_name;
    }
    closedir(dir);
#endif
    free(ptr);
    return 0;
}
```

### 10. 完整代码

#### 整体框架

![image-20231028135200278](https://glf-1309623969.cos.ap-shanghai.myqcloud.com/img/image-20231028135200278.png)



```c
/*
客户端: 浏览器
	- 通过浏览器访问服务器:
		- 访问方式: 服务器的IP地址:端口
	- 应用层协议使用: http, 数据需要在浏览器端使用该协议进行包装
	- 响应消息的处理也是浏览器完成的 => 程序猿不需要管
	- 客户端通过url访问服务器资源
		- 客户端访问的路径:
		1. http://192.168.1.100:8989/  或者  http://192.168.1.100:8989
			- 访问服务器提供的资源目录的根目录
				- 并不是服务器上的 / 目录  
				- 这个目录根据服务器端的描述应该是: /home/robin/luffy 目录
			- 请求行:
				GET / HTTP/1.1
		2. http://192.168.1.100:8989/a.txt
			- 端口后边的/代表服务器的资源根目录
				- 在服务器端路径: /home/robin/luffy 目录
			- 客户端要访问服务器上的a.txt的文件
			- a.txt 这个文件在服务器提供的资源目录中
				- 服务器上的路径: /home/robin/luffy/a.txt
			- 请求行:
				GET /a.txt HTTP/1.1
		3. http://192.168.1.100:8989/hello/a.txt
			- http://192.168.1.100:8989: 服务器地址
			- /hello/a.txt
				- /: 服务器端提供的资源根目录
				- hello: 资源根目录的子目录
				- a.txt: 在hello目录中
			- 请求行:
				GET /hello/a.txt HTTP/1.1
		4. http://192.168.1.100:8989/hello/wrold/
			- http://192.168.1.100:8989: 服务器地址
			- /hello/world/
				- /: 服务器端提供的资源根目录
				- hello: 资源根目录的子目录
				- world/: 如果world后边有/代表这是一个目录, 这个目录在hello目录中
			- 请求行:
				GET /hello/world/ HTTP/1.1
*/

/*
服务器端: 提供服务器, 让客户端访问
	- 支持多客户端访问
		- 使用IO多路转接 => epoll
	- 客户端发送给的请求消息是基于http的
		- 需要能够解析http请求
	- 服务器回复客户端数据, 使用http协议封装回复的数据 ==> http响应
	- 服务器端需要提供一个资源目录, 目录中的文件可以供客户端访问
		- 客户端访问的文件没有在资源目录中, 就不能访问了
			- 假设服务器提供个资源目录: /home/robin/luffy 目录
*/
```

```c
// 服务器端处理的伪代码
int main()
{
    // 1. 创建监听的fd
    socket();
    // 2. 绑定
    bind();
    // 3. 设置监听
    listen();
    
    // 4. 创建epoll模型
    epoll_create();
    epoll_ctl();
    // 5. 检测
    while(1)
    {
        epoll_wait();
        // 监听的文件描述符
        accept();
        // 通信的
        // 接收数据->http请求消息
        recvAndParseHttp();
    }
    return 0;
}

// 基于边沿非阻塞模型接收数据
int recvAndParseHttp()
{
    // 循环接收数据
    // 解析http请求消息
    // http请求由两种:get / post
    // 只处理get请求, 浏览器向服务器请求访问的文件都是静态资源, 因此使用get就可以
    // 判断是不是get请求  ==> 在请求行中 ==> 请求行的第一部分
    // 客户端向服务器请求的静态资源是什么? => 请求行的第二部分
    // 找到服务器上的静态资源
    	- 文件 -> 读文件内容
        - 目录 -> 遍历目录
    // 将文件内容或者目录内容打包到http响应协议中
    // 将整条协议发送回给客户端即可
}
```

#### **epoll_web.c**

```c
#include "epoll_web.h"
#include <arpa/inet.h>
#include <stdio.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <string.h>
#include <strings.h>
#include <errno.h>
#include <sys/stat.h>
#include <assert.h>
#include <sys/sendfile.h>
#include <dirent.h>
#include <unistd.h>
#include <stdlib.h>
#include <ctype.h>
#include <sys/types.h>
// 初始化监听套接字
int initListenFd(unsigned int port){
    // 1. 创建监听套接字
    int lfd = socket(AF_INET,SOCK_STREAM,0);
    if(lfd == -1){
        perror("socket error");
        return -1;
    }
    // 2. 设置端口复用
    int opt = 1;
    int ret = setsockopt(lfd,SOL_SOCKET,SO_REUSEADDR,&opt,sizeof(opt));
    if(ret == -1){
        perror("setsockopt error");
        return -1;
    }
    // 3. 绑定
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = 0;
    ret = bind(lfd,(struct sockaddr *)&addr,sizeof(addr));
    if(ret == -1){
        perror("bind error");
        return -1;
    }
    // 4.设置监听
    ret = listen(lfd,128);
    if(ret == -1){
        perror("listen error");
        return -1;
    }
    // 5. 返回fd
    return lfd;
}

//启动epoll
int epollrun(int lfd){
    // 1. 创建epoll树
    int epfd = epoll_create(1);
    if(epfd == -1){
        perror("epoll_create error");
        return -1;
    }
    // 2. lfd上树
    struct epoll_event ev;
    ev.data.fd = lfd;
    ev.events = EPOLLIN;
    int ret = epoll_ctl(epfd,EPOLL_CTL_ADD,lfd,&ev);
    if(ret == -1){
        perror("epoll_ctl error");
        return -1;
    }
    // 3. 检测(委托内核检测添加到树上的节点)
    struct epoll_event evs[1024];
    int size = sizeof(evs) / sizeof(struct epoll_event);
    while(1){
        int num = epoll_wait(epfd,evs,size,-1);
        if(num == -1) {
            perror("epoll_wait error");
            return -1;
        }
        // 遍历发生变化的节点
        for(int i = 0; i < num; ++i){
            if(!(evs[i].events & EPOLLIN)) {
                // 不是读事件
                continue;
            }
            int fd = evs[i].data.fd;
            if(fd == lfd){
                //建立新连接accept
                accpetClient(lfd,epfd);
            }else{
                // 读数据
                printf("=============before recvHttpRequest=============\n");
                recvHttpRequest(fd,epfd);
                printf("=============after recvHttpRequest=============\n");
            }
        }
    }
    return 0;
}

int accpetClient(int lfd,int epfd){
    // 1. 建立连接
    struct sockaddr_in cliaddr;
    socklen_t len = sizeof(cliaddr);
    cliaddr.sin_family = AF_INET;
    int cfd = accept(lfd,(struct sockaddr*)&cliaddr,&len);
    if(cfd == -1){
        perror("accept error");
        return -1;
    }
    char ip[16]="";
    printf("new client ip=%s port=%d\n",
    inet_ntop(AF_INET, &cliaddr.sin_addr.s_addr,ip,16),ntohs(cliaddr.sin_port));
    
    // 2. 设置cfd为非阻塞
    int flag = fcntl(cfd,F_GETFL);
    flag |= O_NONBLOCK;
    fcntl(cfd,F_SETFL,flag);

    // 3. cfd添加到epoll
    struct epoll_event ev;
    ev.data.fd = cfd;
    // 边沿非阻塞模式
    ev.events = EPOLLIN | EPOLLET;      //边沿模式

    int ret = epoll_ctl(epfd,EPOLL_CTL_ADD,cfd,&ev);
    if(ret == -1){
        perror("epoll_ctl error");
        return -1;
    }

    return 0;
}

int recvHttpRequest(int cfd,int epfd){
    char buf[4096] = { 0 };
    char tmp[1024] = { 0 };
    int len = 0;
    int total = 0;
    // 1. 接收数据
    while((len = recv(cfd,tmp,sizeof(tmp),0)) > 0){
        if(total + len < sizeof(buf)){
            memcpy(buf + total,tmp,len);
        }
        total += len;
    }

    // 2. 判断数据是否接受完毕
    if(len == -1 && errno == EAGAIN){
        // 解析请求行   
        /*
        0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
        G E T   / 1 . t x t H  T  T  P  /  1  .  1  /r /n
        */
        char* pt = strstr(buf,"\r\n");  //大字符串找小字符串
        int reqLen = pt - buf;
        buf[reqLen] = '\0';
        parseRequestLine(buf,cfd);

    }
    else if(len == 0){
        // 客户端断开连接
        epoll_ctl(epfd,EPOLL_CTL_DEL,cfd,NULL);
        close(cfd);
    }
    else{
        perror("recv error");
    }

    return 0;
}

int parseRequestLine(const char* line,int cfd){
    // 1. 拆分http请求行   get /xxx/1.jpg http/1.1
    char method[12];    // 方法
    char path[1024];    // 路径
    char protocol[12];  // 协议
    sscanf(line,"%[^ ] %[^ ] %[^ ]",method,path,protocol);
    printf("method = %s, path = %s, protocol = %s\n", method, path, protocol);

    // 判断是否是get请求
    if(strcasecmp(method,"get") != 0){     //不区分大小写
        return -1;
    }

    // 转码 将不能识别的中文乱码 -> 中文
    // 解码 %23 %34 %5f
    decode_str(path, path);

    // 2. 处理客户端请求的静态资源
    char* file = NULL;
    // 如果没有指定访问的资源, 默认显示资源目录中的内容
    if(strcmp(path,"/") == 0){
        // file的值, 资源目录的当前位置
        file = "./";
    }else{
        // 去掉path中的/ 获取访问文件名
        file = path + 1;
    }
    
    // 3. 获取文件属性
    struct stat st;
    int ret = stat(file,&st);
    if(ret == -1){
        // 文件不存在--回复404
        sendHeadMsg(cfd,404,"Not Found",get_file_type(".html"),-1);
        sendFile("404.html",cfd);
        return 0;
    }
    // 判断文件类型（判断是目录还是文件）
    if(S_ISDIR(st.st_mode)){    // 目录
        // 把目录发给客户端
         sendHeadMsg(cfd,200,"OK",get_file_type(".html"),-1);
         sendDir(file,cfd);
    }else{
        // 把文件内容发给客户端
        sendHeadMsg(cfd,200,"OK",get_file_type(file),st.st_size);
        sendFile(file,cfd);
    }
    return 0;
}

int sendHeadMsg(int cfd,int status,const char* desrc,const char* type,int length){
    // 状态行
    char buf[4096] = { 0 };
    sprintf(buf,"http/1.1 %d %s \r\n",status,desrc);
    // 消息报头
    sprintf(buf + strlen(buf),"Content-Type: %s\r\n",type);
    sprintf(buf + strlen(buf),"Content-Length: %d\r\n",length);

    send(cfd,buf,strlen(buf),0);
     // 空行
    send(cfd, "\r\n", 2, 0);
    return 0;
}

int sendFile(const char* filename,int cfd){

    // 1. 打开文件
    int fd = open(filename,O_RDONLY);
    assert(fd > 0);     // 断言
    // if(fd == -1){
    //     perror("open error");
    // }
    // 2. 循环读文件
#if 1
    char buf[4096] = { 0 };
    int len = 0, ret = 0;
    while((len = read(fd,buf,sizeof(buf))) > 0){
        // 发送读出的数据
        ret = send(cfd,buf,len,0);
        if(ret == -1){
            if(errno = EAGAIN){
                perror("send error:");
                continue;
            }else if (errno == EINTR) {
                perror("send error:");
                continue;
            } else {
                perror("send error:");
                return -1;
            }
        }
    }
#else
    off_t offset = 0;
    int size = lseek(fd,0,SEEK_END);
    lseek(fd,0,SEEK_SET);
    while(offset < size){
        int ret = sendfile(cfd,fd,&offset,size);
        printf("ret value: %d\n",ret);
        if(ret == -1 && errno == EAGAIN){
            printf("没数据。。。\n");
            perror("snedfile");
            
        }
    }
#endif
    close(fd);
    return 0;
}

// 发送目录内容
int sendDir(const char* dirname, int cfd)
{
   
   // 拼接一个html页面<table></table>
   char buf[4096] = { 0 };
   
   sprintf(buf,"<html><head><title>目录名：%s</title></head><body><table>",dirname);
   //sprintf(buf + strlen(buf),"<body><h1>当前目录：%s</h1><table>",dirname);

    // 目录项二级指针
    struct dirent** ptr;
    int num = scandir(dirname,&ptr,NULL,alphasort);

    // 遍历目录
    for(int i = 0; i < num; i++){
        // 取出文件名 namelist 指向的是一个指针数组 struct dirent* tmp[]
        char* name = ptr[i]->d_name;
        char subPath[1024] = { 0 };
        // 拼接文件袋完整路径
        sprintf(subPath,"%s/%s",dirname,name);
        
        struct stat st;
        stat(subPath,&st);

        char enstr[1024] = {0};
        // 编码生成 %E5 %A7 之类的东西
        encode_str(enstr, sizeof(enstr), name);

        // 如果是文件
        if(S_ISREG(st.st_mode)) {       
            sprintf(buf+strlen(buf), 
                    "<tr><td><a href=\"%s\">%s</a></td><td>%ld</td></tr>",
                    enstr, name, (long)st.st_size);
        } else if(S_ISDIR(st.st_mode)) {		// 如果是目录       
            sprintf(buf+strlen(buf), 
                    "<tr><td><a href=\"%s/\">%s/</a></td><td>%ld</td></tr>",
                    enstr, name, (long)st.st_size);
        }
        int ret = send(cfd, buf, strlen(buf), 0);
        if (ret == -1) {
            if (errno == EAGAIN) {
                perror("send error:");
                continue;
            } else if (errno == EINTR) {
                perror("send error:");
                continue;
            } else {
                perror("send error:");
                return -1;
            }
        }
        memset(buf, 0, sizeof(buf));
        // 字符串拼接
        free(ptr[i]);
    }  
    
    // 字符串拼接
    //memset(buf, 0, sizeof(buf));
    sprintf(buf, "</table></body></html>");
    send(cfd, buf, strlen(buf), 0);
    printf("dir message send OK!!!!\n"); 
#if 0
    // 打开目录
    DIR* dir = opendir(dirname);
    if(dir == NULL)
    {
        perror("opendir error");
        exit(1);
    }

    // 读目录
    struct dirent* ptr = NULL;
    while( (ptr = readdir(dir)) != NULL )
    {
        char* name = ptr->d_name;
    }
    closedir(dir);
#endif
    free(ptr);
    return 0;
}

/*
 *  这里的内容是处理%20之类的东西！是"解码"过程。
 *  %20 URL编码中的‘ ’(space)
 *  %21 '!' %22 '"' %23 '#' %24 '$'
 *  %25 '%' %26 '&' %27 ''' %28 '('......
 *  相关知识html中的‘ ’(space)是&nbsp
 */

// 16进制数转化为10进制
int hexit(char c)
{
    if (c >= '0' && c <= '9')
        return c - '0';
    if (c >= 'a' && c <= 'f')
        return c - 'a' + 10;
    if (c >= 'A' && c <= 'F')
        return c - 'A' + 10;

    return 0;
}

void encode_str(char* to, int tosize, const char* from)
{
    int tolen;

    for (tolen = 0; *from != '\0' && tolen + 4 < tosize; ++from) {    
        if (isalnum(*from) || strchr("/_.-~", *from) != (char*)0) {      
            *to = *from;
            ++to;
            ++tolen;
        } else {
            sprintf(to, "%%%02x", (int) *from & 0xff);
            to += 3;
            tolen += 3;
        }
    }
    *to = '\0';
}

void decode_str(char *to, char *from)
{
    for ( ; *from != '\0'; ++to, ++from  ) {     
        if (from[0] == '%' && isxdigit(from[1]) && isxdigit(from[2])) {       
            *to = hexit(from[1])*16 + hexit(from[2]);
            from += 2;                      
        } else {
            *to = *from;
        }
    }
    *to = '\0';
}

// 通过文件名获取文件的类型
const char *get_file_type(const char *name)
{
    char* dot;

    // 自右向左查找‘.’字符, 如不存在返回NULL
    dot = strrchr(name, '.');   
    if (dot == NULL)
        return "text/plain; charset=utf-8";
    if (strcmp(dot, ".html") == 0 || strcmp(dot, ".htm") == 0)
        return "text/html; charset=utf-8";
    if (strcmp(dot, ".jpg") == 0 || strcmp(dot, ".jpeg") == 0)
        return "image/jpeg";
    if (strcmp(dot, ".gif") == 0)
        return "image/gif";
    if (strcmp(dot, ".png") == 0)
        return "image/png";
    if (strcmp(dot, ".css") == 0)
        return "text/css";
    if (strcmp(dot, ".au") == 0)
        return "audio/basic";
    if (strcmp( dot, ".wav" ) == 0)
        return "audio/wav";
    if (strcmp(dot, ".avi") == 0)
        return "video/x-msvideo";
    if (strcmp(dot, ".mov") == 0 || strcmp(dot, ".qt") == 0)
        return "video/quicktime";
    if (strcmp(dot, ".mpeg") == 0 || strcmp(dot, ".mpe") == 0)
        return "video/mpeg";
    if (strcmp(dot, ".vrml") == 0 || strcmp(dot, ".wrl") == 0)
        return "model/vrml";
    if (strcmp(dot, ".midi") == 0 || strcmp(dot, ".mid") == 0)
        return "audio/midi";
    if (strcmp(dot, ".mp3") == 0)
        return "audio/mpeg";
    if (strcmp(dot, ".ogg") == 0)
        return "application/ogg";
    if (strcmp(dot, ".pac") == 0)
        return "application/x-ns-proxy-autoconfig";

    return "text/plain; charset=utf-8";
}
```

#### **epoll_web.h**

```c
#ifndef _EPOLL_SEVER_H
#define _EPOLL_SEVER_H

// 初始化监听的套接字
int initListenFd(unsigned int port);

//启动epoll
int epollrun(int lfd);

// 建立新连接
int accpetClient(int lfd,int epfd);

// 读数据
int recvHttpRequest(int fd,int epfd);

// 解析请求行
int parseRequestLine(const char* line,int cfd);

// 发送响应头（状态行+响应头）
int sendHeadMsg(int cfd,int status,const char* desrc,const char* type,int length);

// 发送文件
int sendFile(const char* filename,int cfd);

// 发送目录
int sendDir(const char* dirName,int cfd);

// 通过文件名获取文件的类型
const char *get_file_type(const char *name);

int hexit(char c);
void encode_str(char* to, int tosize, const char* from);
void decode_str(char *to, char *from);
#endif

```

#### main.c

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include "epoll_web.h"



int main(int argc, char* argv[]){
    if(argc < 3){
        printf("./a.out port path\n");
        exit(1);
    }
    // 采用指定端口
    unsigned int port = atoi(argv[1]);

    // 修改进程工作目录，方便后续操作
    int ret = chdir(argv[2]);
    if(ret == -1){
        perror("chdir error");
    }
    // 初始化监听套接字
    int lfd = initListenFd(port);
    // 启动epoll模型
    epollrun(lfd);
    return 0;
}
```

