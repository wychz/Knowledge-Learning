Handler在netty中，无疑占据着非常重要的地位。Handler与Servlet中的filter很像，通过Handler可以完成通讯报文的解码编码、拦截指定的报文、统一对日志错误进行处理、统一对请求进行计数、控制Handler执行与否。一句话，没有它做不到的只有你想不到的。

Netty中的所有handler都实现自ChannelHandler接口。按照输出输出来分，分为ChannelInboundHandler、ChannelOutboundHandler两大类。ChannelInboundHandler对从客户端发往服务器的报文进行处理，一般用来执行解码、读取客户端数据、进行业务处理等；ChannelOutboundHandler对从服务器发往客户端的报文进行处理，一般用来进行编码、发送报文到客户端。

Netty中，可以注册多个handler。ChannelInboundHandler按照注册的先后顺序执行；ChannelOutboundHandler按照注册的先后顺序逆序执行，如下图所示，按照注册的先后顺序对Handler进行排序，request进入Netty后的执行顺序为：


![img](https://gitee.com/wychz/PicGoBed/raw/master/img/20210318001501)

基本的概念就说到这，下面用一个例子来进行验证。该例子模拟Client与Server间的通讯，Server端注册了2个ChannelInboundHandler、2个ChannelOutboundHandler。当Client连接到Server后，会向Server发送一条消息。Server端通过ChannelInboundHandler 对Client发送的消息进行读取，通过ChannelOutboundHandler向client发送消息。最后Client把接收到的信息打印出来。

Server端一共有5个类：HelloServer InboundHandler1 InboundHandler2 OutboundHandler1 OutboundHandler2

1. HelloServer 代码如下

```java
package com.guowl.testmultihandler;
 
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
 
public class HelloServer {
	public void start(int port) throws Exception {
		EventLoopGroup bossGroup = new NioEventLoopGroup(); 
		EventLoopGroup workerGroup = new NioEventLoopGroup();
		try {
			ServerBootstrap b = new ServerBootstrap(); 
			b.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class) 
					.childHandler(new ChannelInitializer<SocketChannel>() { 
								@Override
								public void initChannel(SocketChannel ch) throws Exception {
									// 注册两个OutboundHandler，执行顺序为注册顺序的逆序，所以应该是OutboundHandler2 OutboundHandler1
									ch.pipeline().addLast(new OutboundHandler1());
									ch.pipeline().addLast(new OutboundHandler2());
									// 注册两个InboundHandler，执行顺序为注册顺序，所以应该是InboundHandler1 InboundHandler2
									ch.pipeline().addLast(new InboundHandler1());
									ch.pipeline().addLast(new InboundHandler2());
								}
							}).option(ChannelOption.SO_BACKLOG, 128) 
					.childOption(ChannelOption.SO_KEEPALIVE, true); 
 
			ChannelFuture f = b.bind(port).sync(); 
 
			f.channel().closeFuture().sync();
		} finally {
			workerGroup.shutdownGracefully();
			bossGroup.shutdownGracefully();
		}
	}
 
	public static void main(String[] args) throws Exception {
		HelloServer server = new HelloServer();
		server.start(8000);
	}
}
```

2. InboundHandler1

```java
package com.guowl.testmultihandler;
 
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class InboundHandler1 extends ChannelInboundHandlerAdapter {
	private static Logger	logger	= LoggerFactory.getLogger(InboundHandler1.class);
 
	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		logger.info("InboundHandler1.channelRead: ctx :" + ctx);
		// 通知执行下一个InboundHandler
		ctx.fireChannelRead(msg);
	}
 
	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		logger.info("InboundHandler1.channelReadComplete");
		ctx.flush();
	}
}
```

3. InboundHandler2

```java
package com.guowl.testmultihandler;
 
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class InboundHandler2 extends ChannelInboundHandlerAdapter {
	private static Logger	logger	= LoggerFactory.getLogger(InboundHandler2.class);
 
	@Override
	// 读取Client发送的信息，并打印出来
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		logger.info("InboundHandler2.channelRead: ctx :" + ctx);
		ByteBuf result = (ByteBuf) msg;
		byte[] result1 = new byte[result.readableBytes()];
		result.readBytes(result1);
		String resultStr = new String(result1);
		System.out.println("Client said:" + resultStr);
		result.release();
 
		ctx.write(msg);
	}
 
	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		logger.info("InboundHandler2.channelReadComplete");
		ctx.flush();
	}
 
}
```

4. OutboundHandler1

```java
package com.guowl.testmultihandler;
 
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelOutboundHandlerAdapter;
import io.netty.channel.ChannelPromise;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class OutboundHandler1 extends ChannelOutboundHandlerAdapter {
	private static Logger	logger	= LoggerFactory.getLogger(OutboundHandler1.class);
	@Override
	// 向client发送消息
	public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
		logger.info("OutboundHandler1.write");
		String response = "I am ok!";
		ByteBuf encoded = ctx.alloc().buffer(4 * response.length());
		encoded.writeBytes(response.getBytes());
		ctx.write(encoded);
		ctx.flush();
	}
	
	
}
```

5. OutboundHandler2

```java
package com.guowl.testmultihandler;
 
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelOutboundHandlerAdapter;
import io.netty.channel.ChannelPromise;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class OutboundHandler2 extends ChannelOutboundHandlerAdapter {
	private static Logger	logger	= LoggerFactory.getLogger(OutboundHandler2.class);
	
	@Override
	public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
		logger.info("OutboundHandler2.write");
		// 执行下一个OutboundHandler
		super.write(ctx, msg, promise);
	}
}
```

Client端有两个类：HelloClient  HelloClientIntHandler

1. HelloClient

  ```java
package com.guowl.testmultihandler;
 
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
 
public class HelloClient {
	public void connect(String host, int port) throws Exception {
		EventLoopGroup workerGroup = new NioEventLoopGroup();
 
		try {
			Bootstrap b = new Bootstrap();
			b.group(workerGroup);
			b.channel(NioSocketChannel.class);
			b.option(ChannelOption.SO_KEEPALIVE, true);
			b.handler(new ChannelInitializer<SocketChannel>() {
				@Override
				public void initChannel(SocketChannel ch) throws Exception {
					ch.pipeline().addLast(new HelloClientIntHandler());
				}
			});
 
			// Start the client.
			ChannelFuture f = b.connect(host, port).sync();
			f.channel().closeFuture().sync();
		} finally {
			workerGroup.shutdownGracefully();
		}
	}
 
	public static void main(String[] args) throws Exception {
		HelloClient client = new HelloClient();
		client.connect("127.0.0.1", 8000);
	}
}
  ```

2. HelloClientIntHandler

```java
package com.guowl.testmultihandler;
 
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class HelloClientIntHandler extends ChannelInboundHandlerAdapter {
	private static Logger	logger	= LoggerFactory.getLogger(HelloClientIntHandler.class);
	@Override
	// 读取服务端的信息
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		logger.info("HelloClientIntHandler.channelRead");
		ByteBuf result = (ByteBuf) msg;
		byte[] result1 = new byte[result.readableBytes()];
		result.readBytes(result1);
		result.release();
		ctx.close();
		System.out.println("Server said:" + new String(result1));
	}
	@Override
	// 当连接建立的时候向服务端发送消息 ，channelActive 事件当连接建立的时候会触发
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		logger.info("HelloClientIntHandler.channelActive");
		String msg = "Are you ok?";
		ByteBuf encoded = ctx.alloc().buffer(4 * msg.length());
		encoded.writeBytes(msg.getBytes());
		ctx.write(encoded);
		ctx.flush();
	}
}
```

server端执行结果为：

![](https://gitee.com/wychz/PicGoBed/raw/master/img/20210318001737.jpg)

在使用Handler的过程中，需要注意：

1. ChannelInboundHandler之间的传递，通过调用 ctx.fireChannelRead(msg) 实现；调用ctx.write(msg) 将传递到ChannelOutboundHandler。

2. ctx.write()方法执行后，需要调用flush()方法才能令它立即执行。

3. ChannelOutboundHandler 在注册的时候需要放在最后一个ChannelInboundHandler之前，否则将无法传递到ChannelOutboundHandler。
   * 在InboundHandler执行完成需要调用OutboundHandler的时候，比如在InboundHandler调用ctx.writeAndFlush()方法，Netty是直接从该InboundHandler返回逆序的查找该InboundHandler之前的OutboundHandler，并非从Pipeline的最后一项Handler开始查找。所以Outbound一定要在最后一个InboundHandler之前。