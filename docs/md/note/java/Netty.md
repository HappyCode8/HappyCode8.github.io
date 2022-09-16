# Netty

## 预备知识

1. buffer的capacity、position、limit、mark

   position指的是下一次读取或写入的位置

   limit指的是最后一位元素的下一位所在的位置

   capacity指的是缓存区最大数据容量

   mark指的是一个备忘位置，调用mark()方法的话，mark值将存储当前position的值，等下次调用reset()方法时，会设定position的值为之前的标记值

   初始化，创建一个7个数组大小的缓存区：

   ![image-20201231122805468](./images/image-20201231122805468.png)

2. buffer的flip、clear、get、put、rewind方法

   put写入3个字节；

   ![image-20201231122845622](./images/image-20201231122845622.png)

   调用flip方法读出三个已写入的数据,limit指向position，position归0：

   ![image-20201231123624375](./images/image-20201231123624375.png)

   调用get一直取数据到3个数据被取完：

   ![image-20201231123803512](./images/image-20201231123803512.png)

   调用clear方法恢复原状：

   ![image-20201231123854719](./images/image-20201231123854719.png)

3. Java的Socket

   服务端：

   ```java
   import java.io.IOException;
   import java.net.ServerSocket;
   
   public class Server {
       private static final String HOST="127.0.0.1";
       private static final Integer PORT=8888;
       public static void main(String[] args) throws IOException {
           final var serverSocket = new ServerSocket(PORT);
           System.out.println("server将一直等待连接的到来");
           final var socket = serverSocket.accept();//一直阻塞等待连接
           final var inputStream = socket.getInputStream();
           byte[] bytes = new byte[1024];
           int len;
           var stringBuilder = new StringBuilder();
           while ((len = inputStream.read(bytes)) != -1) {//一直阻塞等待数据读完, //只有当客户端关闭它的输出流的时候，服务端才能取得结尾的-1
               stringBuilder.append(new String(bytes, 0, len,"UTF-8"));
           }
           System.out.println("获取客户端信息: " + stringBuilder);
           inputStream.close();
           socket.close();
           serverSocket.close();
       }
   }
   ```

   客户端

   ```java
   import java.net.Socket;
   
   public class Client {
       private static final String HOST="127.0.0.1";
       private static final Integer PORT=8888;
       public static void main(String args[]) throws Exception {
           final var socket = new Socket(HOST, PORT);
           final var message="HellloWorld";
           socket.getOutputStream().write(message.getBytes("UTF-8"));
           socket.close();
       }
   }
   ```

4. 双工通信

   ```java
   import java.io.IOException;
   import java.io.OutputStream;
   import java.net.ServerSocket;
   
   public class Server {
       private static final String HOST="127.0.0.1";
       private static final Integer PORT=8888;
       public static void main(String[] args) throws IOException {
           final var serverSocket = new ServerSocket(PORT);
           System.out.println("server将一直等待连接的到来");
           final var socket = serverSocket.accept();
           final var inputStream = socket.getInputStream();
           byte[] bytes = new byte[1024];
           int len;
           var stringBuilder = new StringBuilder();
           while ((len = inputStream.read(bytes)) != -1) {
               stringBuilder.append(new String(bytes, 0, len,"UTF-8"));
           }
           System.out.println("客户端传来的消息: " + stringBuilder);
           OutputStream outputStream = socket.getOutputStream();
           outputStream.write("HelloWorld,too".getBytes("UTF-8"));
           inputStream.close();
           outputStream.close();
           socket.close();
           serverSocket.close();
       }
   }
   ```

   ```java
   import java.io.InputStream;
   import java.net.Socket;
   
   public class Client {
       private static final String HOST="127.0.0.1";
       private static final Integer PORT=8888;
       public static void main(String args[]) throws Exception {
           final var socket = new Socket(HOST, PORT);
           final var message="HellloWorld";
           socket.getOutputStream().write(message.getBytes("UTF-8"));
           socket.shutdownOutput();
           InputStream inputStream = socket.getInputStream();
           byte[] bytes = new byte[1024];
           int len;
           var stringBuilder = new StringBuilder();
           while ((len = inputStream.read(bytes)) != -1) {
               stringBuilder.append(new String(bytes, 0, len,"UTF-8"));
           }
           System.out.println("服务端传来的消息: " + stringBuilder);
   
           inputStream.close();
           socket.close();
       }
   }
   ```

5. 结束符

   <font color='red'>如何告知对方已发送完命令？</font>

   - 通过socket关闭，缺点是不能接受服务端的消息，也不能再次发送

   - 通过socket关闭输出，是关闭socket的输出，而不是关闭输出流，但是也不能再次发送了

     ```
     使用socket.shutdownOutput()关闭而不是outputStream.close();
     ```

   - 通过约定符号，但是符号不好确定，简单符号可能会出现在消息中，复杂符号会占带宽
   - 事先约定长度，在发送消息之前先把消息的长度发送出去

   ```java
   import java.io.IOException;
   import java.net.ServerSocket;
   
   public class Server {
       private static final String HOST="127.0.0.1";
       private static final Integer PORT=8888;
       public static void main(String[] args) throws IOException {
           final var serverSocket = new ServerSocket(PORT);
           System.out.println("server将一直等待连接的到来");
           final var socket = serverSocket.accept();
           final var inputStream = socket.getInputStream();
           byte[] bytes;
           while (true) {
               int first = inputStream.read();// 首先读取两个字节表示的长度
               if(first==-1){//如果读取的值为-1，说明到了流的末尾，Socket已经被关闭了，此时将不能再去读取
                   break;
               }
               int second = inputStream.read();
               int length = (first << 8) + second;
               bytes = new byte[length];// 然后构造一个指定长的byte数组
               inputStream.read(bytes); // 然后读取指定长度的消息即可
               System.out.println("从客户端得到的消息: " + new String(bytes, "UTF-8"));
           }
           inputStream.close();
           socket.close();
           serverSocket.close();
   
       }
   }
   ```

   ```java
   import java.io.OutputStream;
   import java.net.Socket;
   
   public class Client {
       private static final String HOST="127.0.0.1";
       private static final Integer PORT=8888;
       public static void main(String args[]) throws Exception {
           final var socket = new Socket(HOST, PORT);
           OutputStream outputStream = socket.getOutputStream();
           String message = "第一条消息";
           byte[] sendBytes = message.getBytes("UTF-8");
           outputStream.write(sendBytes.length >>8);
           outputStream.write(sendBytes.length);
           outputStream.write(sendBytes);
           outputStream.flush();
           //==========此处重复发送一次
           message = "第二条消息";
           sendBytes = message.getBytes("UTF-8");
           outputStream.write(sendBytes.length >>8);
           outputStream.write(sendBytes.length);
           outputStream.write(sendBytes);
           outputStream.flush();
           //==========此处重复发送一次
           message = "第三条消息";
           sendBytes = message.getBytes("UTF-8");
           outputStream.write(sendBytes.length >>8);
           outputStream.write(sendBytes.length);
           outputStream.write(sendBytes);
   
           outputStream.close();
           socket.close();
       }
   }
   ```

   6. 服务端接收多个请求

   ```java
   import java.io.IOException;
   import java.net.ServerSocket;
   
   public class Server {
       private static final String HOST="127.0.0.1";
       private static final Integer PORT=8888;
       public static void main(String[] args) throws IOException {
           final var serverSocket = new ServerSocket(PORT);
           System.out.println("server将一直等待连接的到来");
           while(true){
               final var socket = serverSocket.accept();
               final var inputStream = socket.getInputStream();
               byte[] bytes = new byte[1024];
               int len;
               final var stringBuilder = new StringBuilder();
               while ((len = inputStream.read(bytes)) != -1) {
                   stringBuilder.append(new String(bytes, 0, len, "UTF-8"));
               }
               System.out.println("从客户端得到的消息: " + stringBuilder);
               inputStream.close();
               socket.close();
           }
       }
   }
   ```

   这种方法耗时，当一个请求需要耗费较多时间的时候，后续的一直得不到处理，优化的方法：

   ```java
   import java.io.IOException;
   import java.net.ServerSocket;
   import java.util.concurrent.ExecutorService;
   import java.util.concurrent.Executors;
   
   public class Server {
       private static final String HOST="127.0.0.1";
       private static final Integer PORT=8888;
       private static final ExecutorService threadPool = Executors.newFixedThreadPool(100);
       public static void main(String[] args) throws IOException {
           final var serverSocket = new ServerSocket(PORT);
           System.out.println("server将一直等待连接的到来");
           while(true){
               final var socket = serverSocket.accept();
               Runnable runnable=()->{
                   try {
                       final var inputStream = socket.getInputStream();
                       byte[] bytes = new byte[1024];
                       int len;
                       var stringBuilder = new StringBuilder();
                       while ((len = inputStream.read(bytes)) != -1) {
                           // 注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
                           stringBuilder.append(new String(bytes, 0, len, "UTF-8"));
                       }
                       System.out.println("从客户端得到的消息: " + stringBuilder);
                       inputStream.close();
                       socket.close();
                   } catch (Exception e) {
                       e.printStackTrace();
                   }
               };
               threadPool.submit(runnable);
           }
       }
   }
   ```

## 阻塞IO缺点

<img src="./images/image-20201231215051523.png" alt="image-20201231215051523" style="zoom:50%;" />

上述代码建立多线程首先，这些线程未必都在用，有资源浪费，其次，每个线程都要求的栈内存从64KB到1MB不等（取决于操作系统），最后即使一个java虚拟机物理上能够支持如此大量的线程，当线程到达1万时也支持不了这儿大的上下文的切换，更何况数量达到10万时。

## 非阻塞IO

<img src="./images/image-20201231220355562.png" alt="image-20201231220355562" style="zoom:50%;" />

使用一个选择器时刻检查哪个Socket准备好了读写，准备好读写的再处理。

## Netty核心概念

四个核心概念，涵盖资源、逻辑以及消息通知，形成了Netty的基于事件的异步IO机制：

- Channels

  到硬件、文件、网络socket、程序组件的连接，能够进行一个或多个IO操作，比如读写，类似于一个搬数据的卡车，它能够打开或关闭，连接与断开。

- Callbacks

  回调，提前注册到适当的地方，合适的时机触发，比如在连接成功时建立一个回调，channelActive方法就会在连接建立成功时触发。

  ![image-20201231224415201](./images/image-20201231224415201.png)

- Futures

  Future是另一种操作完成时的通知方式，这种方式通过通过手动检查是否完成事件，Netty提供了ChannelFuture的实现。

  ![image-20201231224454118](./images/image-20201231224454118.png)

- Events and handlers

  ![image-20201231225123854](./images/image-20201231225123854.png)

## 实现一个EchoServer

目的是将所有收到的数据原样写回去：

<img src="./images/image-20210101103518135.png" alt="image-20210101103518135" style="zoom:50%;" />

### 必要条件

- 至少一个ChannelHandler，负责实现收到客户端数据时的业务逻辑
- Bootstrapping，服务器的启动代码，至少要绑定到端口监听连接

### ChannelHandlers和业务逻辑

- 对到来的数据进行处理，需要实现接口ChanneInboundHandler，这个方法实现了对入栈事件的处理，当我们仅需要其中一些方法时，使用其子类即可ChannelInboundHandlerAdapter。

- 以下方法需要我们关注：

  - ChannelRead，数据到来时的处理
  - ChannelReadComplete，数据传输完成时的处理
  - exceptionCaught，异常时的处理

- SimpleChannelInboundHandle 与 ChannelInboundHandler

  在客户端的业务Handler继承的是SimpleChannelInboundHandler，而在服务器端继承的是ChannelInboundHandlerAdapter。

  最主要的区别就是SimpleChannelInboundHandler在接收到数据后会自动release掉数据占用的Bytebuffer资源(自动调用Bytebuffer.release())。而为何服务器端不能用呢，因为我们想让服务器把客户端请求的数据发送回去，而服务器端有可能在channelRead方法返回前还没有写完数据，因此不能让它自动release。

  ```java
  public void channelRead(ChannelHandlerContext channelHandlerContext, Object msg) {
        final var in=(ByteBuf)msg;
        System.out.println("服务器收到数据："+in.toString(CharsetUtil.UTF_8));
        channelHandlerContext.write(in);//异步，有可能方法返回还没处理完这一部分
    }
  ```

### 服务端

- Handler

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandler.Sharable;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.CharsetUtil;

@Sharable//可以被多个Channel安全共享
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext channelHandlerContext, Object msg) {
        final var in=(ByteBuf)msg;
        System.out.println("服务器收到数据："+in.toString(CharsetUtil.UTF_8));//控制台
        channelHandlerContext.write(in);//写回但是不刷新出站消息
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext channelHandlerContext) {
        channelHandlerContext.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
        //刷新出站消息并且关闭Channel
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext channelHandlerContext, Throwable throwable) {
        throwable.printStackTrace();
        channelHandlerContext.close();
    }
}
```

- server

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

import java.net.InetSocketAddress;

public class EchoServer {
    private static final Integer POTR = 8888;

    public static void main(String[] args) throws Exception {
        new EchoServer().start();
    }

    public void start() throws Exception {
        final var serverHandler = new EchoServerHandler();
        final var group = new NioEventLoopGroup();
        try {
            final var b = new ServerBootstrap();//启动管理
            b.group(group)
                    .channel(NioServerSocketChannel.class)//使用NIO传输通道
                    .localAddress(new InetSocketAddress(POTR))//绑定端口
                    .childHandler(new ChannelInitializer<SocketChannel>() {//绑定处理
                        @Override
                        public void initChannel(SocketChannel socketChannel) {
                            socketChannel.pipeline().addLast(serverHandler);
                        }
                    });
            final var f = b.bind().sync();
            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully().sync();
        }
    }
}
```

### 客户端

- handler

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandler.Sharable;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.util.CharsetUtil;

@Sharable
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {
    @Override
    public void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) {//收到消息时调用
        System.out.println("客户端收到："+byteBuf.toString(CharsetUtil.UTF_8));
    }

    @Override
    public void channelActive(ChannelHandlerContext channelHandlerContex){//连接建立时调用
        channelHandlerContex.writeAndFlush(Unpooled.copiedBuffer("连接建立",CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext channelHandlerContext, Throwable throwable) {
        throwable.printStackTrace();
        channelHandlerContext.close();
    }
}
```

- client

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

import java.net.InetSocketAddress;

public class EchoClient {
    private static final String HOST= "127.0.0.1";
    private static final Integer POTR = 8888;

    public static void main(String[] args) throws Exception {
        new EchoClient().start();
    }
    public void start() throws Exception {
        final var group = new NioEventLoopGroup();
        try {
            final var b = new Bootstrap();//启动管理
            b.group(group)
                    .channel(NioSocketChannel.class)//使用NIO传输通道
                    .remoteAddress(new InetSocketAddress(HOST, POTR))//绑定端口
                    .handler(new ChannelInitializer<SocketChannel>() {//绑定处理
                        @Override
                        public void initChannel(SocketChannel socketChannel) {
                            socketChannel.pipeline().addLast(new EchoClientHandler());
                        }
                    });
            final var f = b.connect().sync();
            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully().sync();
        }
    }
}
```

## Netty组件与设计

### Channel接口

channel接口是对Socket类的基本实现，包括bind、connect、read、write方法

- EmbeddedChannel
- LocalServerChannel
- NioDatagramChannel
- NioSctpChannel
- NioSocketChannel

### EventLoop接口

EventLoop定义了Netty在整个连接的生命周期中处理事件的核心抽象，下图定义了Channels,EventLoops,Threads,EventLoopGroups的关系。

<img src="./images/image-20210101131333156.png" alt="image-20210101131333156" style="zoom:50%;" />

- 一个EventLoopGroup包含一个或多个EventLoop
- 一个EventLoop在整个生命周期中绑定一个线程
- 所有的IO事件都被EventLoop处理
- 一个Channel的整个生命周期都与EventLoop绑定
- 一个EventLoop可能会被一个或多个Channel绑定

根据这个设计，一个确定的Channel的IO事件都是由同一个Thread处理的。

### ChannelFuture接口

ChannelFuture的addListener方法注册了一个ChannelFutureListener监听器，当操作完成时会收到通知。

### ChannelHandler和ChannelPipeline

#### ChannelHandler接口

从开发角度看，channelHandler是Netty的主要组件，它是所有出站入站数据的处理逻辑，这个方法由网络事件触发。channelInboundHandler是它的子接口。

#### ChannelPipeline接口

ChannelPipeline为一系列ChannelHandler提供了一个接口，并且提供了API沿着链路传播出站与入站事件，当一个Channel被创建时，自动分配一个ChannelPipeline如下：

- 通过ServerBootstrap注册一个ChannelInitializer实现
- 当ChannelInitializer.initChannel()被调用时，ChannelInitializer安装一个ChannelHandlers集到pipeline
- ChannelInitializer从ChannelPipeline移除他自己

注意handler对事件的处理取决于它被添加的顺序，从客户端角度来看，如果是从客户端到服务端，就认为是出站，入站则相反。对于入站事件的处理是按照添加顺序进行的，对于出站事件则相反。

<img src="./images/image-20210101140623950.png" alt="image-20210101140623950" style="zoom:50%;" />

### Encoders与Decoders

入站信息会被解码，从字节转换为另一种形式，出站时相反.



​                  