---
title: moduo之Buffer
date: 2024-06-06 16:53:22
tags: moduo
categories:
    笔记
---

# Muduo中的Buffer

`Buffer`类其实是封装了一个用户缓冲区，以及向这个缓冲区写数据读数据等一系列控制方法。

## **为什么需要有应用层缓冲区？**

在非阻塞网络编程中，非阻塞IO的核心思想是**避免阻塞在`read()/write()`或其他IO系统调用上**，可以最大限度复用`thread-of-control`，让一个线程能服务于多个`socket`连接。而IO线程只能阻塞在I/O多路复用函数上，如`select()/poll()/epoll_wait()`，这样应用层的缓冲区就是必须的，每个`TCP socket`都要有足够安全的`input buffer`和`output buffer`。

## Buffer的设计

![image-20240605143814050](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240605143814050.png)

它使用`vector`来作为底层容器，可以进行动态的扩容操作。其次，前置的预留字节空间可用于填充数据序列化后的消息长度。该类的构造函数及其成员变量如下：

```c++
// 网络库底层的缓冲区类型定义
class Buffer {
public:
    static const size_t kCheapPrepend = 8;
    static const size_t kInitialSize = 1024;

    explicit Buffer(size_t initialSize = kInitialSize)
        : buffer_(kCheapPrepend + initialSize),
          readerIndex_(kCheapPrepend),
          writerIndex_(kCheapPrepend) 
    {}private:
 	std::vector<char> buffer_;
    size_t readerIndex_;
    size_t writerIndex_;
}
```

`Buffer`类对于预留空间默认为 8 字节，而其默认大小为 8 + 1024，即 1032 字节。

如下三个函数分别返回预留空间大小、可读空间大小和可写空间大小，而 `peek()` 则返回读指针。

```c++
size_t readableBytes() const { return writerIndex_ - readerIndex_; }
size_t writableBytes() const { return buffer_.size() - writerIndex_; }
size_t prependableBytes() const { return readerIndex_; }
// 返回缓冲区中可读数据的起始地址
const char* peek() const { return begin() + readerIndex_; }
```

如下方法定义了如何从缓冲区中读取数据并更新缓冲区的状态。具体来说，它包括两个函数：`retrieve(size_t len)` 和 `retrieveall()`。

```c++
void retrieve(size_t len) {
    if(len < readableBytes()) {
        readerIndex_ += len;  // 应用只读取了可读缓冲区数据的一部分就是len，还剩下
    } else {
        retrieveall();
    }
}
void retrieveall() {
    readerIndex_ = kCheapPrepend;
    writerIndex_ = kCheapPrepend;
}
```

`retrieve`这个函数的作用是从缓冲区中读取指定长度 `len` 的数据，并根据读取的长度更新 `readerIndex_` 和 `writerIndex_`。`retrieveall`这个函数的作用是重置缓冲区的状态，使得缓冲区变为空。将读取指针和写入指针都重置到缓冲区的起始位置 `kCheapPrepend`。这样，缓冲区变为空，可以重新开始接收和处理新的数据。

具体过程如下图：

![image-20240605144601428](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240605144601428.png)

如下函数：`retrieveAllAsString()` 函数用于将缓冲区中的所有可读数据转换为一个字符串并返回。`retrieveAsString(size_t len)` 函数用于将缓冲区中指定长度的数据转换为一个字符串并返回，同时更新缓冲区的读取指针。

```c++
// 把onMessage函数上报的Buffer数据，转成string类型的数据返回
std::string retrieveAllAsString() {
    return retrieveAsString(readableBytes());
}

std::string retrieveAsString(size_t len) {
    std::string result(peek(), len);
    retrieve(len);  // 对缓冲区做复位操作
    return result;
}
```

如果可写缓冲区中的空间不够写，就需要扩容。`append(const char* data, size_t len)` 函数将给定的 `data` 数据添加到缓冲区中，并更新 `writerIndex_`。

```c++
// buffer_.size - writerIndex_ 表示缓冲区中可写的空间
void ensureWritableBytes(size_t len) {
    if (writableBytes() < len) {
        makeSpace(len);
    }
}

// 把[data, data+len]内存上的数据，添加到writable缓冲区当中
void append(const char* data, size_t len) {
    ensureWritableBytes(len);
    std::copy(data, data + len, beginWrite());
    writerIndex_ += len;
}
```

### 自动增长

这个 `makeSpace(size_t len)` 函数的目的是在缓冲区中腾出足够的空间，以便存储长度为 `len` 的数据。如果当前缓冲区的可写空间不足，函数将通过扩展缓冲区或重新整理现有数据来确保有足够的空间。让我们逐步分析这个函数。

```c++
void makeSpace(size_t len) {
    // 判断可写缓冲区的大小加上预写缓冲区的大小是否小于len加kCheapPrepend
    if (writableBytes() + prependableBytes() < len + kCheapPrepend) {
        // 如果小于，则调整缓冲区的大小
        buffer_.resize(writerIndex_ + len);
    } else {
        // 获取可读大小
        size_t readable = readableBytes();
        // 将从readerIndex_到writerIndex_之间的数据复制到缓冲区的kCheapPrepend位置
        std::copy(begin() + readerIndex_, begin() + writerIndex_, begin() + kCheapPrepend);
        // 修改readerIndex_
        readerIndex_ = kCheapPrepend;
        // 修改writerIndex_
        writerIndex_ = readerIndex_ + readable;
    }
}

```

如果可写区域的长度和预留区域的长度之和要小于所需长度和初始化预留区域的长度(8)之和，就需要对缓冲区进行扩容，其过程如下图所示，可见，扩容后可写区域的长度即为所需长度 len。

![image-20240605150032071](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240605150032071.png)

否则，则表示预留空间和可写区域还很充足，为此需要将预留区域中多出来的部分移动到可写区域中去。其过程如下图所示，先将可写区域的内容向前移动，使得预留区域恢复为初始化大小，即 8。然后移动读指针和写指针。

![image-20240605150055404](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240605150055404.png)



`readFd`用于从文件描述符 `fd` 读取数据，并将其存储在缓冲区中。它使用了 `readv` 函数来一次性读取数据到多个缓冲区。以下是对这段代码的详细解释：

```c++
ssize_t Buffer::readFd(int fd, int* savedErrno)
{
    char extrabuf[65536];
    struct iovec vec[2];
    const size_t writable = writableBytes();
    vec[0].iov_base = begin() + writerIndex_;
    vec[0].iov_len = writable;
    vec[1].iov_base = extrabuf;
    vec[1].iov_len = sizeof extrabuf;
    const int iovcnt = (writable < sizeof extrabuf) ? 2 : 1;
    const ssize_t n = sockets::readv(fd, vec, iovcnt);
    if (n < 0) {
        *savedErrno = errno;
    } else if (implicit_cast<size_t>(n) <= writable) {
        writerIndex_ += n;
    } else {
        writerIndex_ = buffer_.size();
        append(extrabuf, n - writable);
    }
    return n;
}
```

结构体 `iovec` 则用于定义一个向量元素，其定义如下：

```c++
struct iovec {
    void   *iov_base; /* Starting address (内存起始地址）*/
    size_t iov_len;   /* Number of bytes to transfer（这块内存长度） */
};
```

`iov_base` 所指向的缓冲区用于存放网络接受的数据，或者是网络将要发送的数据。而 `iov_len` 字段用于存放接受数据的最大长度，或者是实际写入的数据长度。

在 `readFd` 方法中，定义了两个 `iovec` 结构体，一部分用于将读到的数据放在可写区域，另一部分用于将读到的数据放在额外缓冲区域。然后，使用如下方法进行分散读，即将数据从文件描述符读到分散的内存块中。其中的第三个参数表示` iov` 的数组长度。

```c++
#include <sys/uio.h>
ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
```

从 `readFd` 方法中可以看出，如果可写区域的长度小于 65536，则两个内存块都使用，否则只使用可写区域即可。

最后根据 `readv` 方法的返回值来确定是否需要额外缓冲区，如果返回的字节数要大于可写区域的长度，说明两个内存块都使用了，且可写区域中已经填充满了，其余的数据全部在额外缓冲区中，此时，只需移动写指针到缓冲区的数据末端，然后将额外缓冲区中的数据追加进来即可。

## TcpConnection必须要有output buffer

一个常见场景：程序想通过TCP连接发送100K byte数据，但在write()调用中，OS只接收80K（受TCP通告窗口advertised window的控制），而程序又不能原地阻塞等待，事实上也不知道要等多久。程序应该尽快交出控制器，返回到event loop。此时，剩余20K数据怎么办？

对应用程序，它只管生成数据，不应该关系到底数据是一次发送，还是分几次发送，这些应该由网络库操心，程序只需要调用TcpConnection::send()就行。网络库应该接管剩余的20K数据，把它保存到TcpConnection的output buffer，然后注册POLLOUT事件，一旦socket变得可写就立刻发送数据。当然，第二次不一定能完全写入20K，如果有剩余，网络库应该继续关注POLLOUT事件；如果写完20K，网络库应该停止关注POLLOUT，以免造成busy loop。

如果程序又写入50K，而此时output buffer里还有待发20K数据，那么网络库不应该直接调用write()，而应该把这50K数据append到那20K数据之后，等socket变得可写时再一并写入。

如果output buffer里还有待发送数据，而程序又想关闭连接，但对程序而言，调用TcpConnection::send()后就认为数据迟早会发出去，此时网络库不应该直接关闭连接，而要等数据发送完毕。因为此时数据可能还在内核缓冲区中，并没有通过网卡成功发送给接收方。
将数据append到buffer，甚至write进内核，都不代表数据成功发送给对端。

综上，要让程序在write操作上不阻塞，网络库必须给每个tcp connection配置output buffer。

