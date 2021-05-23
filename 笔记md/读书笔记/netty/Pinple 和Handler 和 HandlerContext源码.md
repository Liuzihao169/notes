## Pinple 、Handler 、 HandlerContext

` ChannelPipeline` 、 `ChannelHandler` 、`ChannelHandlerContext ` 这是netty中的核心组件，他们的关系如下：

1、每当ServerSocket 创建一一个新的连接，就会创建一个Socket, 对应的就是目标客户端。
2、每一个Socket都会分配一个Pipeline 也就是`ChannelPipeline`
3、每个`ChannelPipeline`中有多个Context

4、context之间会形成双向链表在Pipline当中                       

5、而context就是包装之后的Handler

**图示**

![image-20210522213246716](https://gitee.com/liuzihao169/pic/raw/master/image/20210522213248.png)

#### 一、ChannelPipeline

```java
// 可以调用入站 和出站方法，并且实现了迭代器
public interface ChannelPipeline
        extends ChannelInboundInvoker, ChannelOutboundInvoker, Iterable<Entry<String, ChannelHandler>> {
```



进入这个类的源码，doc中提供了一张图：

<img src="https://gitee.com/liuzihao169/pic/raw/master/image/20210523102541.png" alt="image-20210522213950912" style="zoom:50%;" />

上图中体现了Pinple处理的流程：

- pinple内部有多个handler

- 内部分为一个入站handler list和 出站handler list；
- 入站的事件触发为 读事件，而出站的触发事件是写事件

参考我之间的博客 [什么是netty的出站、入站](https://blog.csdn.net/weixin_43732955/article/details/116723951)

![](https://img-blog.csdnimg.cn/img_convert/3c8fdf69fc80d3d57b542c851a97a59b.png)

##### 1.1 何时创建

在socket创建的时候就会创建pinple

```java
  protected AbstractChannel(Channel parent, ChannelId id) {
        this.parent = parent;
        this.id = id;
        unsafe = newUnsafe();
    // 创建pipeline
        pipeline = newChannelPipeline();
    }
```



#### 二、ChannelHandler

`ChannelHandler` 的作用就是拦截IO事件或者处理IO事件，并将其转发给下一个Handler处理，因为处理流转的方向不同，分为入站Handler和出站Handler

<img src="https://gitee.com/liuzihao169/pic/raw/master/image/20210522215558.png" alt="image-20210522215441520" style="zoom:50%;" />

##### 2.1 ChannelInboundHandler

入站Handler主要核心方法如下：

```java
 // channler处于 活跃状态时被调用
void channelActive(ChannelHandlerContext ctx) throws Exception;
// 当从channel中读取数据时被调用
void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception;
```

##### 2.2 ChannelOutboundHandler

出站Handler的核心方法如下：

```java
 // 绑定时调用
void bind(ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise) throws Exception;
void deregister(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;

```

### 三、ChannelHandlerContext

```java
abstract class AbstractChannelHandlerContext implements ChannelHandlerContext, ResourceLeakHint {

    private static final InternalLogger logger = InternalLoggerFactory.getInstance(AbstractChannelHandlerContext.class);
    volatile AbstractChannelHandlerContext next;
    volatile AbstractChannelHandlerContext prev;

    private static final AtomicIntegerFieldUpdater<AbstractChannelHandlerContext> HANDLER_STATE_UPDATER =
            AtomicIntegerFieldUpdater.newUpdater(AbstractChannelHandlerContext.class, "handlerState");

```

* Context 就是包装了handler的相关信息，并且将Handler 通过指针形成双向链表

那么是何时进行包装的呢？找一个pipline的实现类 `DefaultChannelPipeline`

```java
// 节选核心代码
public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
    final AbstractChannelHandlerContext newCtx;
    synchronized (this) {
        checkMultiplicity(handler);
				// 将handler包装成一个context
        newCtx = newContext(group, filterName(name, handler), handler);

        addLast0(newCtx);

    
    callHandlerAdded0(newCtx);
    return this;
}
```

#### 四、小结 

` ChannelPipeline` 、 `ChannelHandler` 、`ChannelHandlerContext `创建流程如下：

- 每当创建ChannelSocket的时候都会创建一个绑定的pipeline，一对一的关系，创建pipeline的时候也会创建tail节点和head节点，形成最初的链表。
- 在调用 pipeline 的addLast 方法的时候，会根据给定的handler 创建一个 Context, 然后，将这个Context 插
  入到链表的尾端(tail 前面)。
-  Context 包装handler， 多个Context 在pipeline 中形成了双向链表
- 入站方向叫 inbound， 由head 节点开始，出站方法叫outbound ，由tail 节点开始

#### 五、ChannelPipeline是如何调度Handler的呢？

`ChannelPipeline`中有一些命名为 `fireXXX`的方法，这些以fire开头的其实都是入站方法，下面根据源码来讲解一下调度流程。以常见的常见的ChannelRead方法调用为例：

```java
   @Override
    public final ChannelPipeline fireChannelRead(Object msg) {
        // 调用invokeXXXX静态方法，并且把head传入，由此可以知道入站是从head开始调用
        AbstractChannelHandlerContext.invokeChannelRead(head, msg);
        return this;
    }
	
```

==AbstractChannelHandlerContext.invokeChannelRead==方法如下：

这个方法调用的是Context的静态方法：

```java
  static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
        final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
        EventExecutor executor = next.executor();
        if (executor.inEventLoop()) {
            // 调用context的invokXXXX()方法
            next.invokeChannelRead(m);
        } else {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    next.invokeChannelRead(m);
                }
            });
        }
    }
```

==invokeChannelRead代码如下==

```java
   private void invokeChannelRead(Object msg) {
        if (invokeHandler()) {
            try {
                //拿到context当中包装的Handler,执行目标的 channelRead方法，
                ((ChannelInboundHandler) handler()).channelRead(this, msg);
            } catch (Throwable t) {
                invokeExceptionCaught(t);
            }
        } else {
            fireChannelRead(msg);
        }
    }
```

加入到了我们定义到 serverHandler `NettyServerHandler` 调用他的 channelRead方法

```java
   @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println(" 服务端接收到客户端的消息>>>>" + byteBuf.toString(CharsetUtil.UTF_8));
        System.out.println(" 服务端接收到客户端的地址为>>>>" + ctx.channel().remoteAddress());
       // 传递到下一个handler
        ctx.fireChannelRead(msg);
    }

```

在这个方法执行完之后，可以调用ChannelHandlerContext的`fireChannelRead()`方法，将调用传递到下一个handler,改方法的代码如下：

==ctx.fireChannelRead(msg==

```java
  @Override
    public ChannelHandlerContext fireChannelRead(final Object msg) {
        // 找到入站handler，执行invokeChannelRead方法，这也就回到了我们第一步的方法，形成了闭环。
        invokeChannelRead(findContextInbound(MASK_CHANNEL_READ), msg);
        return this;
    }
```

##### 调用过程流程如下：

- `ChannelPipeline.fireChannelRead(msg)`方法 --->`AbstractChannelHandlerContext.invokeChannelRead()`方法---

  --->`ChannelInboundHandler.channelRead()`目标方法

  ![image-20210523103640559](https://gitee.com/liuzihao169/pic/raw/master/image/20210523103642.png)

  是否传递到下一个Handler取决于当前handler是否执行了`ctx.fireChannelRead(msg); `