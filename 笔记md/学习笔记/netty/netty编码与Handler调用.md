### 一、编码与解码

1、数据在网络中传输时是以二进制进行传输的，那么业务数据 转成 二进制数据这个过程叫做 编码（encoder），服务端接收数据后，将它转换为业务数据的过程叫做解码（decoder）。

![image-20210512221531113](https://gitee.com/liuzihao169/pic/raw/master/image/20210512221533.png)



### 二、Handler调用链、入站出站

在之前的文档中，介绍netty模型图中 有一个管道ChannelPipline，在这个管道中有一系列HandlerChannelHandler充当了入站和出站的应用处理容器；ChannelPipline 提供了ChannelHandler的容器。

#### 2.1 什么是入站、出站

- ChannelPipline可以理解为一个站台当业务数据进入时叫入站（入站消息会编码）；反之当业务数据出去时称为出站（出站消息会解码）
- Netty中存在一些内置的编码、解码器。这些都是实现了ChannelInboundHandler 或者 ChanneOutboundHandler接口；这些类中的readObject()都被重写，进行编码或解码操作；
- 编码器就是个出站Handler、解码器就是个入栈Handler，而在管道中这些Handler是以有序双向链表的形式存在；

**交互图**

![image-20210512224943307](https://gitee.com/liuzihao169/pic/raw/master/image/20210512224944.png)

注意：ChannelPipline中的Handler实际上一个双向链表，图中为了方便理解画成了两个链表；



### 三、Demo测试

Git代码介绍：https://github.com/Liuzihao169/learn-netty

目录地址 com.io.netty.inAndOutBoundHandle;

- NettyServer.java 服务器端

```java
public class MyServer {
    public static void main(String[] args) throws Exception{

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            /**
             *  自定义一个初始化类
             */
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap
                    .group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();

                            //入站的handler进行解码
                            pipeline.addLast(new MyByteToLongDecoder());
                            //出站的handler进行编码
                            pipeline.addLast(new MyLongToByteEncoder());
                            //自定义的handler
                            pipeline.addLast(new MyServerHandler());
                            System.out.println("xx");
                        }
                    });


            ChannelFuture channelFuture = serverBootstrap.bind(8888).sync();
            channelFuture.channel().closeFuture().sync();

        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }
}
```



- NettyServerHandler.java 服务器端处理器
- NettyClient.java 客户端
- NettyClientHandler.java 客户端处理器

- MyLongToByteEncoder Long编码器

  ```java
   @Override
      protected void encode(ChannelHandlerContext ctx, Long msg, ByteBuf out) throws Exception {
  
          System.out.println("MyLongToByteEncoder encode 被调用;出站编码");
          System.out.println("msg=" + msg);
          out.writeLong(msg);
  
      }
  ```

  

- MyByteToLongDecoder Long解码器

```java
  /**
     *
     * decode 会根据接收的数据，被调用多次, 直到确定没有新的元素被添加到list
     * , 或者是ByteBuf 没有更多的可读字节为止
     * 如果list out 不为空，就会将list的内容传递给下一个 channelinboundhandler处理, 该处理器的方法也会被调用多次
     *
     * @param ctx 上下文对象
     * @param in 入站的 ByteBuf
     * @param out List 集合，将解码后的数据传给下一个handler
     * @throws Exception
     */
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {

        System.out.println("MyByteToLongDecoder 被调用,入站解码");
        //因为 long 8个字节, 需要判断有8个字节，才能读取一个long
        if(in.readableBytes() >= 8) {
            out.add(in.readLong());
        }
    }
```





