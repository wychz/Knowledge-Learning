### **1. BIO, NIO, AIO**

- BIO：一个连接一个线程，客户端有连接请求时服务器端就需要启动一个线程进行处理。线程开销大。
  伪异步IO：将请求连接放入线程池，一对多，但线程还是很宝贵的资源。
- NIO：一个请求一个线程，但客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。
- AIO：一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理，
- BIO是面向流的，NIO是面向缓冲区的；BIO的各种流是阻塞的。而NIO是非阻塞的；BIO的Stream是单向的，而NIO的channel是双向的。
- NIO的特点：事件驱动模型、单线程处理多任务、非阻塞I/O，I/O读写不再阻塞，而是返回0、基于block的传输比基于流的传输更高效、更高级的IO函数zero-copy、IO多路复用大大提高了Java网络应用的可伸缩性和实用性。基于Reactor线程模型。
- 在Reactor模式中，事件分发器等待某个事件或者可应用或个操作的状态发生，事件分发器就把这个事件传给事先注册的事件处理函数或者回调函数，由后者来做实际的读写操作。如在Reactor中实现读：注册读就绪事件和相应的事件处理器、事件分发器等待事件、事件到来，激活分发器，分发器调用事件对应的处理器、事件处理器完成实际的读操作，处理读到的数据，注册新的事件，然后返还控制权。

### **2. Netty介绍**

1. Netty 是一个**基于 NIO**的 client-server(客户端服务器)框架，使用它可以快速简单地开发网络应用程序。
2. 它极大地简化并优化了 TCP 和 UDP 套接字服务器等网络编程,并且性能以及安全性等很多方面甚至都要更好。
3. **支持多种协议**如 FTP，SMTP，HTTP 以及各种二进制和基于文本的传统协议。

**Netty 成功地找到了一种在不妥协可维护性和性能的情况下实现易于开发，性能，稳定性和灵活性的方法。**

### **3. Netty的优点：**

1. 统一的 API，支持多种传输类型，阻塞和非阻塞的。
2. 简单而强大的线程模型。
3. 自带编解码器解决 TCP 粘包/拆包问题。
4. 自带各种协议栈。
5. 真正的无连接数据包套接字支持。
6. 比直接使用 Java 核心 API 有更高的吞吐量、更低的延迟、更低的资源消耗和更少的内存复制。
7. 安全性不错，有完整的 SSL/TLS 以及 StartTLS 支持。

### **4. Netty的应用场景**

1. **作为 RPC 框架的网络通信工具**：我们在分布式系统中，不同服务节点之间经常需要相互调用，这个时候就需要 RPC 框架了。不同服务节点之间的通信是如何做的呢？可以使用 Netty 来做。比如我调用另外一个节点的方法的话，至少是要让对方知道我调用的是哪个类中的哪个方法以及相关参数吧！
2. **实现一个自己的 HTTP 服务器**：通过 Netty 我们可以自己实现一个简单的 HTTP 服务器，这个大家应该不陌生。说到 HTTP 服务器的话，作为 Java 后端开发，我们一般使用 Tomcat 比较多。一个最基本的 HTTP 服务器可要以处理常见的 HTTP Method 的请求，比如 POST 请求、GET 请求等等。
3. **实现一个即时通讯系统**：使用 Netty 我们可以实现一个可以聊天类似微信的即时通讯系统，这方面的开源项目还蛮多的，可以自行去 Github 找一找。
4. **实现消息推送系统**：市面上有很多消息推送系统都是基于 Netty 来做的。

### **5. Netty的核心组件**

1. **Channel**：Channel 接口是 Netty 对网络操作抽象类，它除了包括基本的 I/O 操作，如 bind()、connect()、read()、write() 等。
   比较常用的Channel接口实现类是NioServerSocketChannel（服务端）和NioSocketChannel（客户端），这两个 Channel 可以和 BIO 编程模型中的ServerSocket以及Socket两个概念对应上。Netty 的 Channel 接口所提供的 API，大大地降低了直接使用 Socket 类的复杂性。
2. **EventLoop**：**EventLoop 的主要作用实际就是负责监听网络事件并调用事件处理器进行相关 I/O 操作的处理。**

- Channel 为 Netty 网络操作(读写等操作)抽象类，EventLoop 负责处理注册到其上的Channel 的 I/O 操作，两者配合参与 I/O 操作。



1. **ChannelFuture**：Netty 是异步非阻塞的，所有的 I/O 操作都为异步的。

2. 1. 因此，我们不能立刻得到操作是否执行成功，但是，你可以通过 ChannelFuture 接口的 addListener() 方法注册一个 ChannelFutureListener，当操作执行成功或者失败时，监听就会自动触发返回结果。
   2. 并且，你还可以通过ChannelFuture 的 channel() 方法获取关联的Channel。
   3. 另外，我们还可以通过 ChannelFuture 接口的 sync()方法让异步的操作变成同步的。



1. ChannelHandler：ChannelHandler 是消息的具体处理器。他负责处理读写操作、客户端连接等事情。ChannelPipeline 为 ChannelHandler 的链，提供了一个容器并定义了用于沿着链传播入站和出站事件流的 API 。当 Channel 被创建时，它会被自动地分配到它专属的 ChannelPipeline。

### **6. EventLoopGroup，EventLoop**

![img](C:%5CUsers%5Cwangy%5CDesktop%5C%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93%5CNetty%5Cimg%5CNetty%5Cv2-a0e291a35b5f7b646e1f470defdc6aa5_720w.jpg)



EventLoopGroup 包含多个 EventLoop（每一个 EventLoop 通常内部包含一个线程），上面我们已经说了 EventLoop 的主要作用实际就是负责监听网络事件并调用事件处理器进行相关 I/O 操作的处理。

并且 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理，即 Thread 和 EventLoop 属于 1 : 1 的关系，从而保证线程安全。

上图是一个服务端对 EventLoopGroup 使用的大致模块图，其中 Boss EventloopGroup 用于接收连接，Worker EventloopGroup 用于具体的处理（消息的读写以及其他逻辑处理）。

从上图可以看出：当客户端通过 connect 方法连接服务端时，bossGroup 处理客户端连接请求。当客户端处理完成后，会将这个连接提交给 workerGroup 来处理，然后 workerGroup 负责处理其 IO 相关操作。

- NioEventLoopGroup 默认的构造函数实际会起的线程数为**CPU核心数\*2**

### **7. BootStrap和ServerBootStrap**

1. Bootstrap 通常使用 connet() 方法连接到远程的主机和端口，作为一个 Netty TCP 协议通信中的客户端。另外，Bootstrap 也可以通过 bind() 方法绑定本地的一个端口，作为 UDP 协议通信中的一端。
2. ServerBootstrap通常使用 bind() 方法绑定本地的端口上，然后等待客户端的连接。
3. Bootstrap 只需要配置一个线程组— EventLoopGroup ,而 ServerBootstrap需要配置两个线程组— EventLoopGroup ，一个用于接收连接，一个用于具体的处理。

### **8. Netty线程模型**

> Reactor 模式基于事件驱动，采用多路复用将事件分发给相应的 Handler 处理，非常适合处理海量 IO 的场景。

1. 单线程模型：一个线程需要执行处理所有的 accept、read、decode、process、encode、send 事件。对于高负载、高并发，并且对性能要求比较高的场景不适用。
2. 多线程模型：一个 Acceptor 线程只负责监听客户端的连接，一个 NIO 线程池负责具体处理：accept、read、decode、process、encode、send 事件。满足绝大部分应用场景，并发连接量不大的时候没啥问题，但是遇到并发连接大的时候就可能会出现问题，成为性能瓶颈。
3. 主从多线程模型：从一个 主线程 NIO 线程池中选择一个线程作为 Acceptor 线程，绑定监听端口，接收客户端连接的连接，其他线程负责后续的接入认证等工作。连接建立完成后，Sub NIO 线程池负责具体处理 I/O 读写。如果多线程模型无法满足你的需求的时候，可以考虑使用主从多线程模型 。

![img](C:%5CUsers%5Cwangy%5CDesktop%5C%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93%5CNetty%5Cimg%5CNetty%5Cv2-98e617c474c257d29478d0d024e11d20_720w.jpg)

### **9. 服务端和客户端启动过程**

### **服务端**

1. 首先你创建了两个 NioEventLoopGroup 对象实例：bossGroup 和 workerGroup。

2. 1. bossGroup : 用于处理客户端的 TCP 连接请求
   2. workerGroup ：负责每一条连接的具体读写数据的处理逻辑，真正负责 I/O 读写操作，交由对应的 Handler 处理。



1. 接下来 我们创建了一个服务端启动引导/辅助类：ServerBootstrap，这个类将引导我们进行服务端的启动工作。

2. 通过 .group() 方法给引导类 ServerBootstrap 配置两大线程组，确定了线程模型。

3. 过channel()方法给引导类 ServerBootstrap指定了 IO 模型为NIO

4. 1. NioServerSocketChannel ：指定服务端的 IO 模型为 NIO，与 BIO 编程模型中的ServerSocket对应
   2. NioSocketChannel : 指定客户端的 IO 模型为 NIO， 与 BIO 编程模型中的Socket对应



1. 通过 .childHandler()给引导类创建一个ChannelInitializer ，然后制定了服务端消息的业务处理逻辑 HelloServerHandler 对象
2. 调用 ServerBootstrap 类的 bind()方法绑定端口

### **客户端**

1. 创建一个 NioEventLoopGroup 对象实例
2. 创建客户端启动的引导类是 Bootstrap
3. 通过 .group() 方法给引导类 Bootstrap 配置一个线程组
4. 通过channel()方法给引导类 Bootstrap指定了 IO 模型为NIO
5. 通过 .childHandler()给引导类创建一个ChannelInitializer ，然后制定了客户端消息的业务处理逻辑 HelloClientHandler 对象
6. 调用 Bootstrap 类的 connect()方法进行连接，这个方法需要指定两个参数：host，port

### **10. TCP沾包/拆包**

TCP 粘包/拆包 就是你基于 TCP 发送数据的时候，出现了多个字符串“粘”在了一起或者一个字符串被“拆”开的问题。

解决方案：

1. **使用 Netty 自带的解码器**

2. 1. **LineBasedFrameDecoder**: 发送端发送数据包的时候，每个数据包之间以换行符作为分隔。LineBasedFrameDecoder 的工作原理是它依次遍历 ByteBuf 中的可读字节，判断是否有换行符，然后进行相应的截取
   2. **DelimiterBasedFrameDecoder**: 可以自定义分隔符解码器，
   3. **LineBasedFrameDecoder**实际上是一种特殊的 DelimiterBasedFrameDecoder 解码器。
   4. **FixedLengthFrameDecoder**: 固定长度解码器，它能够按照指定的长度对消息进行相应的拆包。
   5. **LengthFieldBasedFrameDecoder**



1. **自定义序列化编解码器**

在 Java 中自带的有实现 Serializable 接口来实现序列化，但由于它性能、安全性等原因一般情况下是不会被使用到的。

通常情况下，我们使用 Protostuff、Hessian2、json 序列方式比较多，另外还有一些序列化性能非常好的序列化方式也是很好的选择：

- 专门针对 Java 语言的：Kryo，FST 等等
- 跨语言的：Protostuff（基于 protobuf 发展而来），ProtoBuf，Thrift，Avro，MsgPack 等等

### **11. 长连接和心跳机制**

在 TCP 保持长连接的过程中，可能会出现断网等网络异常出现，异常发生的时候， client 与 server 之间如果没有交互的话，它们是无法发现对方已经掉线的。为了解决这个问题, 我们就需要引入**心跳机制**。

心跳机制的工作原理是: 在 client 与 server 之间在一定时间内没有数据交互时, 即处于 idle 状态时, 客户端或服务器就会发送一个特殊的数据包给对方, 当接收方收到这个数据报文后, 也立即发送一个特殊的数据报文, 回应发送方, 此即一个 PING-PONG 交互。所以, 当某一端收到心跳消息后, 就知道了对方仍然在线, 这就确保 TCP 连接的有效性.

TCP 实际上自带的就有长连接选项，本身是也有心跳包机制，也就是 TCP 的选项：SO_KEEPALIVE。但是，TCP 协议层面的长连接灵活性不够。所以，一般情况下我们都是在应用层协议上实现自定义心跳机制的，也就是在 Netty 层面通过编码实现。通过 Netty 实现心跳机制的话，核心类是 IdleStateHandler 。

### **12. Netty的零拷贝机制**



![img](C:%5CUsers%5Cwangy%5CDesktop%5C%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93%5CNetty%5Cimg%5CNetty%5Cv2-d79d0ffc734ddd70686ba7d4e2e197ce_720w.jpg)

![img](C:%5CUsers%5Cwangy%5CDesktop%5C%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93%5CNetty%5Cimg%5CNetty%5Cv2-5a241cc038753fab5c784a1aabe4a2cc_720w.jpg)



从图中可以看出文件经历了4次copy过程：

1.首先，调用read方法，文件从user模式拷贝到了kernel模式；（用户模式->内核模式的上下文切换，在内部发送sys_read() 从文件中读取数据，存储到一个内核地址空间缓存区中）

2.之后CPU控制将kernel模式数据拷贝到user模式下；（内核模式-> 用户模式的上下文切换，read()调用返回，数据被存储到用户地址空间的缓存区中）

3.调用write时候，先将user模式下的内容copy到kernel模式下的socket的buffer中（用户模式->内核模式，数据再次被放置在内核缓存区中，send（）套接字调用）

4.最后将kernel模式下的socket buffer的数据copy到网卡设备中；（send套接字调用返回）

从图中看2，3两次copy是多余的，数据从kernel模式到user模式走了一圈，浪费了2次copy。应用程序只是起到缓存数据被将传回到套接字的作用而已，别无他用。

数据可以直接从read buffer 读缓存区传输到套接字缓冲区，也就是省去了将操作系统的read buffer 拷贝到程序的buffer，以及从程序buffer拷贝到socket buffer的步骤，直接将read buffer拷贝到socket buffer。JDK NIO中的的`transferTo()` 方法就能够让您实现这个操作，这个实现依赖于操作系统底层的sendFile（）实现

![img](C:%5CUsers%5Cwangy%5CDesktop%5C%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93%5CNetty%5Cimg%5CNetty%5Cv2-1b56f9c606dc92377773117b2e066b64_720w.jpg)

![img](C:%5CUsers%5Cwangy%5CDesktop%5C%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93%5CNetty%5Cimg%5CNetty%5Cv2-e340135d7218043454f11a0c0dab6c72_720w.jpg)



1.transferTo()方法使得文件的内容直接copy到了一个read buffer（kernel buffer）中

2.然后数据（kernel buffer）copy到socket buffer中

3.最后将socket buffer中的数据copy到网卡设备（protocol engine）中传输；

这个显然是一个伟大的进步：这里上下文切换从4次减少到2次，同时把数据copy的次数从4次降低到3次；但是从上述过程中发现从kernel buffer中将数据copy到socket buffer是没有必要的；

![img](C:%5CUsers%5Cwangy%5CDesktop%5C%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93%5CNetty%5Cimg%5CNetty%5Cv2-4294b165efa3d1ce5f796e9c7384ea2c_720w.jpg)



改进后的处理过程如下：

1. 将文件拷贝到kernel buffer中；(DMA引擎将文件内容copy到内核缓存区)
2. 向socket buffer中追加当前要发生的数据在kernel buffer中的位置和偏移量；
3. 根据socket buffer中的位置和偏移量直接将kernel buffer的数据copy到网卡设备（protocol engine）中；

从图中看到，linux 2.1内核中的 “**数据被copy到socket buffe**r”的动作，在Linux2.4 内核做了优化，取而代之的是只包含关于数据的位置和长度的信息的描述符被追加到了socket buffer 缓冲区中。**DMA引擎直接把数据从内核缓冲区传输到协议引擎**（protocol engine），从而消除了最后一次CPU copy。经过上述过程，数据只经过了2次copy就从磁盘传送出去了。这个才是真正的Zero-Copy







在 OS 层面上的 Zero-copy 通常指避免在 用户态(User-space) 与 内核态(Kernel-space) 之间来回拷贝数据。而在 Netty 层面 ，零拷贝主要体现在对于数据操作的优化。

Netty 中的零拷贝体现在以下几个方面：

[https://www.cnblogs.com/xys1228/p/6088805.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/xys1228/p/6088805.html)

1. 使用 Netty 提供的 **CompositeByteBuf** 类, 可以将多个ByteBuf 合并为一个逻辑上的 ByteBuf, 避免了各个 ByteBuf 之间的拷贝。
2. 通过wrap操作实现零拷贝。通过 `Unpooled.wrappedBuffer` 方法来将 bytes 包装成为一个 UnpooledHeapByteBuf 对象, 而在包装的过程中, 是不会有拷贝操作的. 即最后我们生成的生成的 ByteBuf 对象是和 bytes 数组共用了同一个存储空间, 对 bytes 的修改也会反映到 ByteBuf 对象中.
3. ByteBuf 支持 **slice** 操作, 因此可以将 ByteBuf 分解为多个共享同一个存储区域的 ByteBuf, 避免了内存的拷贝。
4. Java NIO 的 `FileChannel` 实现零拷贝, 可以直接将源文件的内容直接拷贝(`transferTo`) 到目的文件中, 而不需要额外借助一个临时 buffer, 避免了不必要的内存操作
   Netty第一步是通过 `RandomAccessFile` 打开一个文件, 然后 Netty 使用了 `DefaultFileRegion` 来封装一个 `FileChannel`.当有了 FileRegion 后, 我们就可以直接通过它将文件的内容直接写入 Channel 中, 而不需要像传统的做法: 拷贝文件内容到临时 buffer, 然后再将 buffer 写入 Channel.                                   