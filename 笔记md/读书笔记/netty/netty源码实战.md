### 一、Netty是什么

Netty 是一个利用 Java 的高级网络的能力，隐藏其背后的复杂性而提供一个易于使用的 API 的客户端/服务器框架。

> 本质：网络应用程序框架
>
> 实现：异步、事件驱动
>
> 特性：高性能、可维护、快熟开发
>
> 用途：开发服务器和客户端。
>
> 框架：dubbo

#### 1.1为什么不使用NIO

> 支持常用应用层协议
>
> 解决传输的问题：粘包、半包现象
>
> 支持流量整形
>
> 完善的断连、ldle等异常处理

#### 1.3 I/o模型

io模型的理解：就是用什么样的通道进行数据发送和接收，很大程度上决定了通信性能。

BIo: 阻塞io 模型，一个连接进行一个服务的处理，如果没有结束会一直等待

Nio: 非阻塞io模型，一个线程可以处理多个请求io,客户端发送的请求会注册到多路复用器上，多路复用就一个线程检查连接的状态

AIo: 异步i o 服务端装备好数据后，会主动通知客户端拿获取数据

#### 1.4 NIO 存在的问题

1、NIO的类库和API繁杂，使用麻烦:需要熟练掌握Selector、 ServerSocketChannel、SocketChannel、 ByteBuffer

2、需要具备其他的额外技能:要熟悉Java多线程编程，因为NIO编程涉及到Reactor 模式，你必须对多线程.和网络编程非常熟悉，才能编写出高质量的NIO程序。

3、开发工作量和难度都非常大:例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常流的处理等等。

4、JDKNIO 的Bug: 例如臭名昭著的Epoll Bug,它会导致Selector 空轮询,最终导致CPU 100%。直到JDK 1.7
版本该问题仍旧存在，没有被根本解决。

### 二、线程模型

#### 2.1 传统I/O 阻塞模型

特点：

一个请求过来，一个线程连接处理；采用阻塞式IO获取输入数据；

如图：

![image-20210509085934194](https://gitee.com/liuzihao169/pic/raw/master/image/20210509085937.png)

缺点：线程数多，服务器压力大，造成资源浪费；阻塞式io，线程利用率不高；

#### 2.2 Reactor模式

基于传统i/o 模型的缺点，有两种解决方案：

1、基于IO复用模型：多个连接共用一个阻塞对象，应用程序只需要在**一个阻塞对象等待**，无需阻塞等待所有连接。当某个连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理。

2、基于线程池复用模型：不必再为每个连接创建线程，将连接完成后的业务处理任务分配给线程进行处理，
一个线程可以处理多个连接的业务。

**Reactor模式就是： I/O复用 + 线程池**模型 如下图：

![image-20210509094517688](https://gitee.com/liuzihao169/pic/raw/master/image/20210509094522.png)

Reactor模式是基于事件驱动，应用程序收到不同的事件后，分发给不同的事件进行处理；**Reactor模式采用I/O复用监听事件，然后分发处理，这是netty高性能的关键**

- （转发handler）reactor 负责分发 
- handler 负责处理

##### 2.2.1 根据reactor模式的分类

a.单reactor 单线程模式  b.单reactor多线程模式。c.主从reactor多线程模式

###### a.单reactor单线程

![image-20210509110037496](https://gitee.com/liuzihao169/pic/raw/master/image/20210509110040.png)

流程说明：

如果是建立连接请求：则交给Acceptor来进行连接处理；否者创建一个handler进行处理；

缺点：单个线程性能比较差



###### b.单reactor多线程

![image-20210509110143213](https://gitee.com/liuzihao169/pic/raw/master/image/20210509110149.png)

流程说明：

如果是建立连接请求：则交给Acceptor来进行连接处理；否者创建一个handler进行处理,但是handler不进行业务处理，只负责接收和响应，真正的业务操作会交给Worka线程来进行处理。

缺点：所有的响应和发送都是Reactor进行，在高并发场景容易出现性能瓶颈



###### c.多线程多reactor模式

![image-20210509110226226](https://gitee.com/liuzihao169/pic/raw/master/image/20210509110229.png)

流程说明：

**主要区别就是 主reactor负责连接，从reactor负责请求处理，然后分发；**

1) Reactor 主线程MainReactor 对象通过select监听连接事件,收到事件后，通过Acceptor处理连接事件
2)当Acceptor处理连接事件后，MainReactor将连接分配给SubReactor
3)
subreactor将连接加入到连接队列进行监听，并创建handler进行各种事件处理
4)当有新事件发生时，subreactor就会调用对应的handler处理
5) handler 通过read读取数据，分发给后面的worker线程处理
6)worker线程池分配独立的worker线程进行业务处理，并返回结果

#### 2.3 小结 reactor模式的优点

- 响应快，不必为单个同步时间所阻塞，虽然Reactor本身依然是同步的
- 可以最大程度的避免复 杂的多线程及同步问题，并且避免了多线程/进程的切换开销
- 扩展性好，可以方便的通过增加Reactor 实例个数来充分利用CPU资源
- 复用性好， Reactor 模型本身与具体事件处理逻辑无关，具有很高的复用性

### 三、Netty模型

netty模型主要是基于 主从多线程reactor模型，简易模型如下：

![image-20210509113032203](https://gitee.com/liuzihao169/pic/raw/master/image/20210509113041.png)

**上图工作流程**

1、BossGroup 线程维护Selector，只关注Accecpt
2、当 接收到Accept事件，获取到对应的SocketChannel,封装成NIOScoketChannel 并注册到Worker线程(事件循环)，并进行维护
3、当Worker线程监听到selector中通道发生自己感兴趣的事件后，就进行处理(就由handler)，注意handler已经加入到通道

#### 3.1 netty的Reactor模式图解

![image-20210509123810418](https://gitee.com/liuzihao169/pic/raw/master/image/20210509123816.png)

1.Netty有两组线程池**BossGroup**专门负责接收客户端的连接，**WorkerGroup**专门负责网络的读写。类型都是NioEventIoopGroup
2.NioEventLoopGroup 相当于一一个事件循环组，这个组中含有多个事件循环，每一个事件循环是NioEventLoop .
3.NioEventLoop 表示一个不断循环的执行处理任务的线程，每个NioEventLoop都有一个selector,用于监听绑定在其上的socket的网络通讯

Boss NioEventLoop循环执行的步骤有3步：

- 轮询accept 事件、
- 处理accept事件，与 client建立连接，生 成NioScocketChannel，并将其注册到某个worker NIOEventLoop、
- 处理任务队列的任务，即runAllTasks

Worker NIOEventLoop循环也是3步骤：

- 轮询read, write事件、

- 处理i/o事件，即 read，write事件，在对应NioScocketChannel处理
- 处理任务队列的任务，即runAllTasks

4.每个Worker NIOEventLoop处 理业务时，会使用pipeline(管道), pipeline中包含了channel ，即通过pipeline
可以获取到对应通道，管道中维护了很多的处理器.

### 四、netty Demo 实践

代码介绍：

- NettyServer.java 服务器端
- NettyServerHandler.java 服务器端处理器
- NettyClient.java 客户端
- NettyClientHandler.java 客户端处理器



**NettyServer.java**

```java
/**
 * @author liuzihao
 * @create 2021-05-09-15:24
 * netty客户端
 */
public class NettyServer {

    public static void main(String[] args) throws Exception{

        /**
         * 创建两个线程组合 bossGroup workGroup 如上图所介绍
         * bossGroup; 只负责连接请求
         * workGroup; 真正和客户段业务处理
         */
        EventLoopGroup bossGroup = new NioEventLoopGroup();

        EventLoopGroup workGroup = new NioEventLoopGroup();

        // 创建服务器的启动参数配置类
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        try {

            serverBootstrap.group(bossGroup, workGroup)
                    // 指定类型为 NioServerSocketChannel
                    .channel(NioServerSocketChannel.class)
                    // 设置线程队列连接个数 108
                    .option(ChannelOption.SO_BACKLOG, 108)
                    // 设置保持连接活动状态
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    // 初始化业务handler 如图处理业务到handler
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            // 往管道加入业务处理handler
                            ch.pipeline().addLast(new NettyServerHandler());
                        }
                    });

            System.out.println("服务器端初始化完成 ....");

            // 绑定端口 获取异步对象
            ChannelFuture sync = serverBootstrap.bind(8888).sync();

            // 异步监听关闭通道
            sync.channel().closeFuture().sync();

        }finally {
            // 发生异常优雅关闭
            bossGroup.shutdownGracefully();
        }
    }
}

```

**NettyServerHandler.java**

```java
/**
 * @author liuzihao
 * @create 2021-05-09-15:24
 * 服务端业务处理
 */
public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * 从通道中读取数据；获取客户端发送的数据
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println(" 服务端接收到客户端的消息>>>>" + byteBuf.toString(CharsetUtil.UTF_8));
        System.out.println(" 服务端接收到客户端的地址为>>>>" + ctx.channel().remoteAddress());


    }

    /**
     * 获取数据完成
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        System.out.println(" 服务端 处理完客户端的消息>>>>并进行回复");
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello im doing ", CharsetUtil.UTF_8));
    }

    /**
     * 发生异常之后 直接关闭
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

**NettyClient.java**

```java
/**
 * @author liuzihao
 * @create 2021-05-09-15:24
 * netty 客户端
 */
public class NettyClinet {

    public static void main(String[] args) throws Exception{

        // 客户端只需要一个 group
        EventLoopGroup group = new NioEventLoopGroup();

        // 设置相关属性
        Bootstrap bootstrap = new Bootstrap();
        try {

            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {

                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new NettyClinetHandler());
                        }
                    });

            System.out.println("客户端 准备好了...");

            ChannelFuture sync = bootstrap.connect("127.0.0.1", 8888).sync();

            // 异步监听关闭通道
            sync.channel().closeFuture().sync();

        }finally {
            group.shutdownGracefully();
        }
    }
}
```

**NettyClinetHandler**

```java
/**
 * @author liuzihao
 * @create 2021-05-09-15:25
 * 客户端 handler
 */
public class NettyClinetHandler extends ChannelInboundHandlerAdapter {

    /**
     * 通道连接之后
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("客户端发送端数据>>>>>>hello server !!!!", CharsetUtil.UTF_8));
    }

    /**
     * 读取通道的数据
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println(" 客户端接收服务端的消息>>>>" + byteBuf.toString(CharsetUtil.UTF_8));
        System.out.println(" 客户端接收服务端的消息>>>>" + ctx.channel().remoteAddress());
    }

    /**
     * 发生异常
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```

### 五、Netty的Task

3种使用的经典场景：

1、用户通过定义自定义任务添加到 Evenloop当中

2、用户自定义定时任务

3、非当前reactor线程调用线程的Channel的各种方法

### 六、Netty异步模型

1、异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的组件在
完成后，通过状态、通知和回调来通知调用者。
2、Netty 中的I/O 操作是异步的，包括Bind、 Write、 Connect 等操作会简单的返回一个ChannelFuture。
3、调用 者并不能立刻获得结果，而是通过Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得I0操作结果。
4、Netty 的异步模型是建立在future 和callback 的之上的。

- callback 就是回调。

- Future， 它的核心思想: 假设一个方法fun, 计算过程可能非常耗时，等待fun 返回显然不合适。那么可以在调用fun 的时候，立
  马返回一个Future， 后续可以通过Future 去监控方法fun的处理过程(即: Future-Listener 机制)

### 七、编解码器与Handler调用机制

1、数据在网络中传输时是以二进制进行传输的，那么业务数据 转成 二进制数据这个过程叫做 编码（encoder），服务端接收数据后，将它转换为业务数据的过程叫做解码（decoder）。

2、ChannelHandler充当了入站和出站的应用处理容器；ChannelPipline 提供了ChannelHandler的容器，

- Channel可以理解为一个站台当业务数据进入时叫入站（入站消息会编码）；反之当业务数据出去时称为出站（出站消息会解码）
- Netty中存在一些内置的编码、解码器。这些都是实现了ChannelInboundHandler 或者 ChanneOutboundHandler接口；这些类中的readObject()都被重写，进行编码或解码操作；



### 八、TCP粘包和拆包

1、TCP是面向连接的，面向流的，提供高可靠性服务。收发两端(客户端和服务器端)都要有- - -成对的socket,因此，发送端为了将多个发给接收端的包，更有效的发给对方，使用了优化方法(Nagle 算法)，将多次间隔较小且数据量小的数据，合并成-一个 大的数据块，然后进行封包。这样做虽然提高了效率，但是接收端就难于分辨出完整的数据包了，因为面向流的通信是无消息保护边界的





