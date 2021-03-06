---
layout: post
title: Netty生命周期
date: 2016-11-16 10:20:00
tags:
- Atom
categories: Text Editor
---

Netty中的关键对象有：
* Channel
* Pipeline
* ChannelHandlerContext
* Handler
* Bootstrap
* ServerBootstrap

这节要讨论的问题主要是Netty中这些关键对象的生命周期。以及这些对象是什么级别的，或者这样问，一个连接中只有一个响应的实例，还是所有连接中只有一个实例，还是每来一个消息都会生成一个相应的实例。如果这些问题不弄清楚，对开发会有影响。

> 如果Netty的底层实现没弄清楚，是不太敢用Netty的。


# Channel

|           Status        |                                  desc                              |                       handler method                |
| ----------------------- | ------------------------------------------------------------------ | --------------------------------------------------- |
| channelUnregistered     | channel已创建但没注册到一个EventLoop                                 | ChannelInboundHandler.channelUnregistered()         | 
| channelRegistered       | channel已注册到一个EventLoop                                        | ChannelInboundHandler.channelRegistered()           |
| channelActive           | channel变为活跃状态(已连接到远程主机)，现在可以接受和发送数据了         | ChannelInboundHandler.channelActive()               |
| channelInactive         | channel处于非活跃状态，没有连接到远程主机                             | ChannelInboundHandler.channelInactive()             |    


```text
+---------------------+          +---------------------+
|                     |          |                     |
|  ChannelRegistered  +--------> |  ChannelActive      |
|                     |          |                     |
+---------------------+          +----------+----------+
                                            |
                                            |
                                            v
+---------------------+          +---------------------+
|                     |          |                     |
| ChannelUnregistered | <--------+ ChannelInactive     |
|                     |          |                     |
+---------------------+          +---------------------+

```

# ChannelHandler

|         Status      |                    Desc                |
| ------------------- | -------------------------------------- |
| handlerAdded        | ChannelHandler添加到ChannelPipeline     |
| handlerRemoved      | ChannelHandler从ChannelPipeline移除     |
| exceptionCaught     | 当ChannelPipeline执行抛出异常时          |




