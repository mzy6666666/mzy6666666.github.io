---
title: 腾讯后端开发QQ部门一面
date: 2024-04-23 20:32:24
tags: 后端开发 C++
categories:
    面经
---

# 腾讯面经

## 进程间通信方式

1. **管道（Pipe）：** 管道是一种半双工的通信方式，用于在父进程和子进程之间传递数据。它只能在具有亲缘关系的进程之间使用，例如父子进程或者兄弟进程。管道可以是匿名的，也可以是有名字的。
2. **命名管道（Named Pipe）：** 命名管道是一种特殊的管道，允许在无亲缘关系的进程之间进行通信。它通过文件系统中的特殊文件来实现，具有持久性和独立性的特点。
3. **消息队列（Message Queue）：** 消息队列是一种通过消息传递方式进行进程间通信的机制。它允许一个进程向队列中发送消息，另一个进程从队列中接收消息，实现了异步、解耦合和灵活性的通信方式。
4. **信号量（Semaphore）：** 信号量是一种用于控制多个进程对共享资源的访问的机制。它通过计数器来实现，允许多个进程同时访问共享资源，但可以限制同时访问的进程数量。
5. **共享内存（Shared Memory）：** 共享内存是一种高效的进程间通信方式，允许多个进程共享同一块物理内存区域。它通过将内存映射到进程的地址空间中来实现，使得多个进程可以直接访问共享的数据，而无需进行复制或者传输。
6. **套接字（Socket）：** 套接字是一种用于在网络中进行进程间通信的接口。它允许不同主机上的进程之间通过网络进行数据交换，实现了跨主机的进程通信。
7. **信号（Signal）：** 信号是一种用于通知进程发生了某个事件的机制。它可以在进程之间进行简单的异步通信，例如进程可以通过发送信号来通知另一个进程发生了某个事件，另一个进程可以通过信号处理函数来处理这个事件。

## tcp三次握手、四次挥手

一开始，客户端和服务端都处于 `CLOSE` 状态。先是服务端主动监听某个端口，处于 `LISTEN` 状态。

- 客户端会随机初始化序号（`client_isn`），将此序号置于 TCP 首部的「序号」字段中，同时把 `SYN` 标志位置为 `1`，表示 `SYN` 报文。接着把第一个 SYN 报文发送给服务端，表示向服务端发起连接，该报文不包含应用层数据，之后客户端处于 `SYN-SENT` 状态。
- 服务端收到客户端的 `SYN` 报文后，首先服务端也随机初始化自己的序号（`server_isn`），将此序号填入 TCP 首部的「序号」字段中，其次把 TCP 首部的「确认应答号」字段填入 `client_isn + 1`, 接着把 `SYN` 和 `ACK` 标志位置为 `1`。最后把该报文发给客户端，该报文也不包含应用层数据，之后服务端处于 `SYN-RCVD` 状态。

- 客户端收到服务端报文后，还要向服务端回应最后一个应答报文，首先该应答报文 TCP 首部 `ACK` 标志位置为 `1` ，其次「确认应答号」字段填入 `server_isn + 1` ，最后把报文发送给服务端，这次报文可以携带客户到服务端的数据，之后客户端处于 `ESTABLISHED` 状态。
- 服务端收到客户端的应答报文后，也进入 `ESTABLISHED` 状态。

三次握手的**首要原因是为了防止旧的重复连接初始化造成混乱。**

## protobuf和json的优缺点

**优势**

- **极高的效率：** `ProtoBuf`以二进制格式存储数据，比文本格式的`JSON`小得多，从而减少了传输时间和存储空间。
- **强大的数据类型支持：** `ProtoBuf`支持各种数据类型，包括基本类型、枚举、消息和可重复字段，使其非常适合处理复杂的数据结构；**无类型**：`JSON`是一种无类型的数据格式，因此在数据校验和一致性方面不如`protobuf`。
- **强大的序列化和反序列化性能：** `ProtoBuf`提供了高效的序列化和反序列化库，可以快速将数据转换为二进制格式，并在需要时将其转换回原始对象。

**劣势**

- **语言无关性：** `JSON`是一种语言无关的数据格式，可以轻松地与各种编程语言集成。
- **可读性差**：由于是二进制格式，不易人类阅读和调试。`JSON`是一种文本格式，易于人类阅读和编写。

## 异步日志为什么要用单例模式

1. **全局唯一性：** 异步日志通常需要在整个程序中被访问和使用，因此需要确保日志实例的全局唯一性，以避免多个日志实例之间的冲突和混乱。单例模式确保了在整个程序中只有一个日志实例存在。
2. **方便访问：** 使用单例模式可以通过静态方法或者全局函数来获取日志实例，从而方便在程序的任何地方使用相同的日志实例进行记录，而无需传递日志实例的引用。
3. **线程安全性：** 异步日志通常会在多个线程中被同时使用，因此需要考虑线程安全性。单例模式可以在实现时考虑线程安全性，并确保日志实例的创建和访问都是线程安全的。

## static修饰的变量和全局变量有什么区别

1. **作用域**:
   - 全局变量在整个程序中都是可见的，可以被任何函数访问。
   - `static`修饰的变量只在定义它的函数内部可见，其作用域被限制在定义它的函数内部。
2. **生存周期**:
   - 全局变量在程序运行期间始终存在，直到程序结束。
   - `static`修饰的变量也在程序运行期间存在，但其生命周期与程序中对应的函数调用有关。当包含`static`变量的函数执行完毕时，`static`变量仍然存在，但其值会保持上一次函数调用结束时的状态。
3. **存储位置**:
   - 全局变量存储在静态存储区，程序在启动时就会为全局变量分配内存。
   - `static`修饰的变量也存储在静态存储区，但只有在**其所在的函数第一次被调用时才会分配内存**。
4. **访问权限**:
   - 全局变量可以被程序中的任何函数访问。
   - `static`修饰的变量只能被定义它的函数访问，其他函数无法直接访问该变量。

## epoll的底层原理

`epoll` 是 Linux 下的一种 I/O 多路复用机制，用于处理大量并发连接或操作。它通过操作系统内核提供的 `epoll` 系统调用来管理文件描述符并监视 I/O 事件。

1. **事件驱动模型：** `epoll` 是基于**事件驱动**的模型。它利用操作系统内核的事件通知机制来通知应用程序发生了哪些 I/O 事件，例如套接字可读、套接字可写等。
2. **注册事件：** 应用程序可以通过 `epoll_ctl` 系统调用向内核注册需要监视的文件描述符以及对应的事件类型（读、写等）。一旦文件描述符上发生了指定的事件，内核就会通知应用程序。
3. **等待事件：** 应用程序使用 `epoll_wait` 系统调用来等待事件的发生。当有文件描述符上发生了注册的事件时，`epoll_wait` 将返回并通知应用程序。
4. **高效处理事件：** `epoll` 的高效性体现在它在等待事件发生时不会阻塞整个进程，而是采用了非阻塞的方式。它通过内核态和用户态之间的数据结构共享，避免了频繁的上下文切换，从而提高了性能。
5. **可扩展性：** `epoll` 在设计上考虑了可扩展性，能够有效地处理大量的并发连接或操作。它使用红黑树（Red-Black Tree）和双链表（Doubly Linked List）等数据结构来管理文件描述符，从而减少了对文件描述符数量的限制。

## 什么情况下需要I/O多路复用技术

传统的 I/O 模型中，每个连接或操作都需要一个独立的线程或进程来处理，这会消耗大量的系统资源。而使用 I/O 多路复用技术，可以通过一个线程或进程同时监听多个 I/O 事件，从而减少了资源的消耗。

**IO多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态**，比如调用select和poll函数，传入多个文件描述符，**如果有一个文件描述符就绪，则返回，否则阻塞直到超时**。

这样在处理1000个连接时，**只需要1个线程监控就绪状态，对就绪的每个连接开一个线程处理就可以了，这样需要的线程数大大减少，减少了内存开销和上下文切换的CPU开销。**

## tcp三次握手期间，ddos攻击

“分布式拒绝服务”，即利用大量合法的分布式服务器对目标发送请求，从而导致正常合法用户无法获得服务。

通过发送伪造源`IP`的TCP数据包发送`SYN`或`ACK`包、发送包含错误设置的地址值等攻击方式，**耗尽服务器资源，导致服务器拒绝访问**。

## 在没有IO多路复用的时候，早年的网络服务器，如PHP写的他们是怎么处理连接的，采用什么技术

**多进程/多线程模型：** 服务器启动时创建多个进程或线程来处理连接。每个连接都被分配给一个独立的进程或线程来处理，这样可以同时处理多个连接。

**阻塞式 I/O：**当有新连接到来时，服务器将为其分配一个进程或线程，并在该进程或线程中使用阻塞式 I/O 来处理连接。

**进程池/线程池：** 为了避免频繁地创建和销毁进程或线程带来的开销，服务器可能会使用进程池或线程池来管理连接处理的进程或线程。

## 算法

## KMP

```c++
void getNext(const string &T, vector<int> &next){
	int j = 0;
	next[0] = 0;
	for(int i = 1; i < T.size(); i++){
		while(j != 0 && T[i] != T[j]){
            // 找前一位的对应的回退位置
			j = next[j-1];
		}
		if(s[i] == s[j]){
			j++;
		}
		next[i] = j;
	}
}
int KMP(string &S ,string &T){
    int n = S.size();
    int m = T.size();
    vector<int> next(m,0);
    getNext(T,next);
    int j = 0;
    for(int i = 0; i < n; i++){
        while(j > 0 && S[i] != T[j]){
            j = next[j-1];
        }
        if(S[i] == T[j]){
            j++;
        }
        if(j == m){
            return (i - m + 1);
        }
    }
    return -1;
}
```

## 快排

```c++
/*
参数说明：a --待排序的数组
		l --数组的左边界
		r --数组的右边界
*/
void quick_sort(vector<int>& a,int l, int r){
    if(l < r){
        int i,j,k;
        i = l;
        j = r;
        k = a[i];
        while(i < j){
            while(i < j && k > a[j]){
                j--;	// 从右向左找第一个大于k的数
            }
            if(i < j){
                a[i++] = a[j];
            }
            while(i < j && k < a[i]){
                i++;	// 从左向右找第一个小于k的数
            }
            if(i < j){
                a[j--] = a[i];
            }
        }
        a[i] = k;
        quick_sort(a,l,i-1);	//递归调用
        quick_sort(a,i+1,r);	//递归调用
    }
}
```

## LRU

```c++
struct Node{
    int key,value;
    Node* prev;
    Node* next;
    Node(int k = 0, int v = 0) :key(k),value(v) {}
};
class LRUCache {
private:
    int capacity;
    Node *dummy;
    unordered_map<int,Node*> key_to_node;
    void remove(Node *x){
        x->prev->next = x->next;
        x->next->prev = x->prev;
    }
    void push_front(Node *x){
        x->prev = dummy;
        x->next = dummy->next;
        x->prev->next= x;
        x->next->prev = x;
    }
    Node *get_node(int key){
        auto it = key_to_node.find(key);
        if(it == key_to_node.end()){
            return nullptr;
        }
        auto node = it->second;
        remove(node);
        push_front(node);
        return node;
    }
public:
    LRUCache(int capacity)
     : capacity(capacity),
     dummy(new Node()) 
    {
        dummy->prev = dummy;
        dummy->next = dummy;
    }
    
    int get(int key) {
        auto node = get_node(key);
        return node ? node->value:-1;
    }
    
    void put(int key, int value) {
        auto node = get_node(key);
        if(node){
            node->value = value;
            return ;
        }
        key_to_node[key] = node = new Node(key,value);
        push_front(node);
        if(key_to_node.size() > capacity){
            auto back_node = dummy->prev;
            key_to_node.erase(back_node->key);
            remove(back_node);
            delete back_node;
        }
    }
};

```

## top K

```c++
vector<int> topKFrequent(vector<int>& nums,int k){
    struct cmp{
        bool operator()(const pair<int,int>& p1, const pair<int,int>& p2){
            return p1.second > p2.second;
        }
	}
	// 统计频率
    unordered_map<int,int> mp;
    for(auto i:nums){
        mp[i]++;
    }
    // 定义堆
    priority_queue<pair<int,int>,vector<pair<int,int>>,cmp> pq;
    for(auto i:mp){
        pq.push(i);
        if(pq.size() > k){
            pq.pop();
        }
    }
    vector<int> res;
    while(!pq.empty()){
        res.push_back(pq.top().first);
        pq.pop();
    }
    return res;
}


```

```c++
static bool cmp(pair<int, int>& m, pair<int, int>& n)  {
       return m.second > n.second;
}
priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(&cmp)> q(cmp);
/*
这里使用了decltype关键词（since c++11）并传入cmp函数的地址，所以第三个模板参数的类型实际上是一个静态函数指针，于是q实例化了一个该模板类的类型出来，但这也只是声明出了一个函数指针而已。

根据priority_queue的构造函数explicit priority_queue (const Compare& comp = Compare(), const Container& ctnr = Container())，通过q(cmp)将cmp这个函数指针赋值给之前在类中声明出的函数指针，从而完成优先队列q的构造。
*/
```

