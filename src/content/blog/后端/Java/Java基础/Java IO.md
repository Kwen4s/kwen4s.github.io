---
title: 'Java IO'
description: 'Java IO 学习笔记'
pubDate: 'Sep 29 2026'
tags: ['Java', 'IO']
---

## 一、Java IO 流的分类？字节流和字符流有什么区别？

### 1. 按操作的数据单位分类

- **字节流（Byte Stream）**：以字节为单位读写数据，基类是 `InputStream` 和 `OutputStream`，可以处理图片、视频、音频和文本等各种类型的文件。
- **字符流（Character Stream）**：以字符为单位读写数据，基类是 `Reader` 和 `Writer`，主要用于处理文本文件，内部会进行字符集的解码和编码。

### 2. 按流向分类

- **输入流**：将数据从外部读取到程序中，例如 `InputStream` 和 `Reader`。
- **输出流**：将程序中的数据写出到外部，例如 `OutputStream` 和 `Writer`。

### 3. 按角色分类

- **节点流**：直接连接数据源，例如 `FileInputStream`，负责与文件、内存或网络等数据源建立连接。
- **处理流（包装流）**：不直接连接数据源，而是包装节点流来增强功能，例如 `BufferedInputStream` 可以提供缓冲读写能力。

## 二、BIO、NIO、AIO 有什么区别？

| 维度 | BIO（Blocking I/O） | NIO（Non-blocking I/O） | AIO（Asynchronous I/O） |
| --- | --- | --- | --- |
| 中文名称 | 同步阻塞 I/O | 同步非阻塞 I/O（New I/O） | 异步非阻塞 I/O |
| 数据模型 | 面向流（Stream） | 面向缓冲区（Buffer） | 面向缓冲区 |
| 线程模型 | 一连接一线程：连接建立后通常需要分配一个线程处理，线程空闲时也会阻塞等待 | 多路复用：一个线程可以管理多个连接，没有数据可读时不会阻塞在线程上 | 异步回调：操作系统完成 I/O 后主动通知应用程序，不需要线程持续等待 |
| 适用场景 | 连接数较少且固定的传统 Web 应用 | 连接数较多、连接时间较短的聊天、推送和网关服务 | 连接数较多、连接时间较长且 I/O 操作较重的场景 |
| 类比 | 服务员站在桌边等待客人点餐 | 服务员先记录需求，再去服务其他客人，餐点准备好后再处理 | 客人下单后由后厨完成，完成后主动通知客人取餐 |

### 简单记忆

- **BIO**：一个连接通常对应一个线程，线程会一直等。
- **NIO**：一个线程可以管理多个连接，只处理已经准备好的连接。
- **AIO**：应用发起 I/O 后就可以继续做其他事情，I/O 完成后再接收通知。

## 三、NIO 的三大核心组件是什么？

NIO 的核心在于 `Buffer`、`Channel` 和 `Selector` 三大组件的协同工作：

1. **Buffer（缓冲区）**：用于存放数据的容器。NIO 中的数据通常需要先读入 Buffer，再进行处理。核心属性包括：
   - `capacity`：容量，表示缓冲区最多可以存放多少数据。
   - `limit`：上限，表示当前最多可以读写到哪个位置。
   - `position`：当前位置，表示下一次读写数据的位置。
   - `mark`：标记位置，可以通过 `reset()` 返回到该位置。
2. **Channel（通道）**：数据传输的双向通道，与只能单向传输的传统 Stream 不同。常见实现有 `FileChannel` 和 `SocketChannel`。
3. **Selector（选择器）**：NIO 的多路复用器，用于监听注册在其上的多个 Channel 事件，例如读就绪和写就绪。当某个 Channel 准备好进行 I/O 操作时，Selector 才会通知程序处理它。

可以简单理解为：`Channel` 负责传输数据，`Buffer` 负责暂存数据，`Selector` 负责监听多个 Channel 是否已经准备好。

### NIO 是怎么实现的？

NIO 是一种同步非阻塞 I/O 模型，也可以理解为非阻塞 I/O。这里的“同步”是指线程需要主动查询 I/O 事件是否就绪；“非阻塞”是指线程等待 I/O 的过程中，可以继续处理其他连接或任务。

它的核心实现过程如下：

1. 将 `Channel` 配置为非阻塞模式，并注册到 `Selector`，告诉 Selector 需要关注连接建立、数据读取或数据写入等事件。
2. 事件循环线程调用 `Selector.select()`，等待已经准备好的 I/O 事件。没有事件就绪时，线程会在 Selector 上等待，而不是为每个连接单独阻塞一个线程。
3. 当某个 Channel 有事件就绪时，Selector 返回对应的 `SelectionKey`，线程根据事件类型处理连接、读取或写入操作。
4. 读取数据时，Channel 将数据写入 `Buffer`；写出数据时，程序将数据放入 `Buffer`，再由 Channel 发送出去。

因此，一个线程就可以监听多个 Channel，只有真正准备好进行 I/O 的连接才会被处理。耗时的业务逻辑通常还会交给工作线程池执行，避免阻塞 Selector 的事件循环线程。

```text
Channel 注册到 Selector
          ↓
Selector 监听 I/O 事件
          ↓
返回已经就绪的 Channel
          ↓
Channel 与 Buffer 进行数据读写
          ↓
继续监听下一批事件
```

## 四、有哪些框架使用了 NIO？

最常见的框架是 **Netty**。Netty 是一个基于 Java NIO 的网络应用框架，常用于网络服务器、RPC、即时通信、网关和长连接服务。

### 1. Netty 如何使用 NIO？

Netty 对 Java NIO 做了封装，把 `Channel`、`Selector`、线程池和网络事件处理组织在一起，让开发者不需要直接编写复杂的 NIO 事件循环代码。

一个请求的大致处理流程是：

```text
客户端建立连接
        ↓
Boss EventLoop 接收连接（accept）
        ↓
Worker EventLoop 监听读写事件
        ↓
ChannelPipeline 依次处理事件
        ↓
解码 → 业务处理 → 编码
        ↓
将结果写回客户端
```

- **Boss EventLoop**：负责接收客户端连接，并将连接分配给 Worker EventLoop。
- **Worker EventLoop**：负责监听连接上的读写事件，一个 EventLoop 可以管理多个 Channel。
- **ChannelPipeline**：事件处理链，负责按照顺序执行解码器、业务处理器和编码器。
- **业务线程池**：如果业务处理比较耗时，可以交给独立线程池执行，避免阻塞 I/O 事件循环。

### 2. Netty 与 Selector、epoll 的关系

- 在 Java NIO 模式下，Netty 底层通常使用 `Selector` 监听多个 `Channel` 的 I/O 事件。
- 在 Linux 环境中，Netty 还可以使用基于 `epoll` 的本地传输实现，减少事件监听和线程调度的开销。
- 无论使用 `Selector` 还是 `epoll`，核心思想都是：少量 EventLoop 线程管理大量连接，只处理已经准备好进行读写的连接。

### 3. Reactor 和 Proactor

- **Reactor**：应用程序主动监听 I/O 是否就绪，事件就绪后再执行读写操作。Java NIO 和 Netty 通常采用这种模型。
- **Proactor**：应用程序发起异步 I/O，操作系统完成读写后通知应用程序处理结果，更接近 AIO 的思想。

因此，学习 Netty 前可以先记住：**Netty 封装了 NIO 的 Channel、Selector 和线程模型，通过 EventLoop 监听事件，再通过 Pipeline 分发和处理网络数据。**
