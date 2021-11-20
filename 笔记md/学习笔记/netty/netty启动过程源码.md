## netty启动原理

代码案例：

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

##### 一、创建NioEventLoopGroup

` NioEventLoopGroup`、和`NioEventLoopGroup` 是netty中的核心对象，前者用于建立连接，注册channel；后者是真正与客户端交流

new NioEventLoopGroup() 如果不指定会默认创建 cup核数 * 2 个线程进行处理；

```java
// cpu核心数 乘以 2
DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
                "io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));

// 创建线程组 部分代码
  children = new EventExecutor[nThreads];

        for (int i = 0; i < nThreads; i ++) {
            boolean success = false;
            try {
              //  其实就是 NioEventLoopGroup对象 （顶层都是EventExecutor接口）
                children[i] = newChild(executor, args);
            }
```



##### 二、创建ServerBootstrap

ServerBootstrap是netty的引导类，是进行服务的启动和参数的初始化。

==serverBootstrap.group(bossGroup, workGroup)==将循环事件组赋值给引导类:

```java

public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup) {
        super.group(parentGroup);
        if (this.childGroup != null) {
            throw new IllegalStateException("childGroup set already");
        }
        this.childGroup = ObjectUtil.checkNotNull(childGroup, "childGroup");
        return this;
    }
```

==.channel(NioServerSocketChannel.class)== 设置一个Channel类信息

```java
  // 根据class 创建生成channel的 Factory 后面会根据这个工厂 创建对应的Channel
public B channel(Class<? extends C> channelClass) {
        return channelFactory(new ReflectiveChannelFactory<C>(
                ObjectUtil.checkNotNull(channelClass, "channelClass")
        ));
    }
```

==.option(ChannelOption.SO_BACKLOG, 108)== 设置tcp参数

```java
// option是一个LinkedHashMap 存储相关配置
public <T> B option(ChannelOption<T> option, T value) {
        ObjectUtil.checkNotNull(option, "option");
        synchronized (options) {
            if (value == null) {
                options.remove(option);
            } else {
                options.put(option, value);
            }
        }
        return self();
    }
```



==.childHandler(Handler)== 设置一个处理的Handler

```java
 // handler属性赋值 会在每次客户端连接的时候调用
public ServerBootstrap childHandler(ChannelHandler childHandler) {
        this.childHandler = ObjectUtil.checkNotNull(childHandler, "childHandler");
        return this;
    }

```



##### 三、端口绑定 serverBootstrap.bind(8888).sync();

这个方法的作用就是创建NioServerSocketChannel，并设置相关属性，然后进行绑定；

```java
//部分代码
private ChannelFuture doBind(final SocketAddress localAddress) {
       // 初始化 和注册 channel
        final ChannelFuture regFuture = initAndRegister();
        final Channel channel = regFuture.channel();
        if (regFuture.cause() != null) {
            return regFuture;
        }

        if (regFuture.isDone()) {
            // At this point we know that the registration was complete and successful.
            ChannelPromise promise = channel.newPromise();
          // 绑定操作
            doBind0(regFuture, channel, localAddress, promise);
            return promise;
```

==initAndRegister()方法==

```java
// 部分代码
    final ChannelFuture initAndRegister() {
        Channel channel = null;
        try {
          // channelFactory 创建出 Channel对象，这个工厂就是 channel(NioServerSocketChannel.class) 所创建的工厂
          // 这里用于创建具体的 Channel
            channel = channelFactory.newChannel();
          
          // 进行初始化 主要是进行初始化，内部操作如下：
          //1、为channel设置TCP属性
          //2、将handler添加到pipeline的末尾（在tail之前），并且会创建一个context对象将handler 与 pipeline进行关联
            init(channel);
          
           // 注册 将Channel注册到EventGroup当中
            ChannelFuture regFuture = config().group().register(channel);

    
```

==doBind0(regFuture, channel, localAddress, promise);== 

```java
// 这bind最终会调用到 NioServerSocketChannel到 doBind方法上 底层使用的是NIO
  protected void doBind(SocketAddress localAddress) throws Exception {
        if (PlatformDependent.javaVersion() >= 7) {
            javaChannel().bind(localAddress, config.getBacklog());
        } else {
            javaChannel().socket().bind(localAddress, config.getBacklog());
        }
    }
```

