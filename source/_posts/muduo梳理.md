---
title: muduo梳理
date: 2024-06-08 17:22:54
tags: muduo
categories:
    笔记
---

# `Muduo`梳理

## 使用`Muduo`搭建简单的服务器

1. 建立事件循环器`EventLoop : loop`
2. 建立服务器对象`TcpServer ：server`
3. 向`TcpServer`中注册各类事件的用户自定义的回调函数：`onConnection`,`onMessage`
4. 设置底层线程数量
5. 启动`start`函数，`server.start()`
6. 开启主线程（`mainloop`)事件循环`loop.loop()`

## 构建TCP对象

在`TcpServer`的构造函数中建立一个`Acceptor`的对象，在`Acceptor`的构造函数中创建了一个非阻塞的`listenfd`，并封装为`Channel`对象`acceptChannel`。在`Acceptor`构造函数中，给`acceptChannel`设置了一个`ReadCallback`回调，事件绑定了`handleread`。当有新用户连接到来时，`Poller`监听`channel`发生事件了，然后上报给`EventLoop`，通知`channel`处理相应的事件注册。在底层`Channel`中的`handleEvent`就会监控到`EPOLLIN`事件，并执行相应的`readCallback()`；而对于`acceptChannel`对应的`readCallback()`方法，就是执行`Accpetor::handleread()`，实际上是执行一个`newConnectionCallback_()`的回调，而`newConnectionCallback_()`是通过`setnewConnectionCallback`设置的，而`Acceptor`是由`TcpServer`管理的，实际是设置的`setnewConnectionCallback`是`TcpServer::newConnection`



> `acceptChannel`注册在事件监听器`Poller`上，`acceptChannel`发生可读事件（即新用户连接）就会调用 `Acceptor::handleRead `
>
> `Acceptor::setReadCallback` => `Acceptor::handleRead `=> `newConnectionCallback`
>
> ​																											||
>
> ​																`TcpServer`的构造函数中设置了`acceptor->setnewConnectionCallback`
>
> ​																											||
>
> ​																       实际上设置的就是`TcpServer::newConnection`
>
> 注册在`mainloop`中的`acceptor`，当有新用户连接到来，最终相应的是`TcpServer::newConnection`

而`TcpServer::newConnection()`实际是创建了一个`TcpConnection`的对象

## `start()`方法调用

>`threadPool_->start(threadInitCallback_);` // 启动底层的loop线程池
>
>​							||
>
>创建`loop`子线程并开启`loop.loop()`（子线程）
>
>执行`accept::listen()`方法，实际上就是将`accpetChannel`注册在`mainloop`的`Poller`上

## 开启`mainloop`的`loop`（为搭建服务器的最后一步)

## 新用户连接到来

>`TcpServer::newConnection`
>
>​				||
>
>根据轮询算法，选择一个`subLoop`，即`ioloop`
>
>创建了一个`TcpConnection`对象，注册相应的回调（`close`的回调对应`TcpServer::removeConnection`）
>
>直接调用`TcpConnection::connectEstablished`
>
> ` ioLoop->runInLoop` => `TcpConnection::connectEstablished`,因为设置了子线程数量（如果没有设置子线程数量，之间在`mainloop`中执行），`runInLoop`实际调用的是`queueInLoop()`，执行`wakeup()`，通过`ioloop`的指针找到`wakeupfd`，向`wakeupfd`写一个数据，`wakeupChannel`就发生读事件，当前`loop`线程(即`ioloop`)就会被唤醒，唤醒后执行`connectEstablished()`。
>
>​				||
>
>`TcpConnection::connectEstablished`
>
>向`poller`注册`channel`的`epollin`事件，然后执行`connectionCallback`，对应的就是用户自定义的`onConnection`函数执行成功，就会有`socket`和`Channel`注册在这个`subloop`上

## 连接成功后，数据通信

> 成功建立连接后
> 		||
> 有可读事件到来，就会执行`Channel`中的`readCallback()`，而`readCallback()`是通过`setReadCallback`设置的，在`TcpConnection`中，`setReadCallback`绑定是`TcpConnection::handleRead`
>
> ​		||
>
> 开始读数据，读完数据后，执行用户自定义的回调操作`onMessage`，然后就得到原始字符串，进行业务处理，业务处理完进行响应（如：回送消息）

## 连接关闭

>通信异常或者对端关闭，底层`channel`都会触发`closeCallback`的回调，而`closeCallback()`是通过`setCloseCallback`设置的。在`TcpConnection`中，`setCloseCallback`绑定是`TcpConnection::handleClose`
>
>​													||
>
>`channel_->disableAll()`，将`chanel`所有感兴趣的事件，在`poller`上全部进行删除，然后执行用户的回调
>
>执行`closeCallback_(connPtr)`,关闭连接的回调  执行的是`TcpServer::removeConnection`回调方法
>
>​													||
>
>​	将保存所有的`TcpConnection`连接的`map`删除，然后获取这条连接对应的`ioloop`，然后执行`TcpConnection::connectDestroyed`，把`channel`从`poller`中删除掉





