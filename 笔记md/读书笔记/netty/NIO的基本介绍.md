#### 一、什么是NIO

- Java NIO全称java non-blocking IO， 是指JDK提供的新API。从JDK1.4开始，Java提供了一系列改进的输入/输出的新特性，被统称为NIO(即New IO)，是同步非阻塞的

-  NIO有三大核心部分: **Channel(通道)， Buffer(缓冲区),Selector(选择器)**

- NIO是面向缓冲区，或者面向块编程的。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络。

#### 二、NIO 与BIO 模型对比

> BIO 是同步阻塞IO,服务器的模式是一个线程处理一个请求，当无响应时，会阻塞线程

> NIO 同步非阻塞IO,会有一个Selector管理多个线程，当有事件发生后，进行处理、不会发生阻塞(是IO多路复用模型)

![image-20210502094415803](https://gitee.com/liuzihao169/pic/raw/master/image/20210502094420.png)

#### 三、NIO 与BIO的差异

1、BIO 以流的方式处理数据,而NIO以块的方式处理数据,块I/O 的效率比流I/O高很多
2、BIO 是阻塞的，NIO则是非阻塞的
3、 BIO基 于字节流和字符流进行操作，而NIO 基于Channel(通道)和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择器)用于监听多个通道的事件(比如:连接请求，数据到达等)，因此使用单个线程就可以监听多个客户端通道

#### 四、什么是Buffer(缓冲区)

**缓冲区(Buffer)**  ： 缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个容器对象(含数组)，该对象提供了一组方法，可以更轻松地使用内存块，，缓冲区对内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由Buffer。

##### 在buffer类中都有4个属性

| Capacity | 容量，即可以容纳的最大数据量                                 |
| -------- | ------------------------------------------------------------ |
| Limit    | 表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作。且极限是可以修改的 |
| Position | 位置，下一个要被读或写的元素的索引，                         |
| Mark     | 标记                                                         |

#### 五、什么是Channel(通道)

NIO的通道类似与流但是又有区别：通道可以进行读写，而流只能读或者写；通道可以支持异步读写。

#### 六、什么是Selector(选择器)

Selector能够检测多个注册的通道上是否有事件发生(**多个Channel以事件的方式可以注册到同一个Selector**)

- 如果有事件发生，便获取事件(通过selectKey)然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求。

- 只有在连接真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程， 不用去维护多个线程避免了多线程之间的上下文切换导致的开销

**核心流程：**

1、注册通道  2、监听通道 3、获取通道 执行业务逻辑

#### 案例

**服务端**

```java
public class NIOServer {

    public static void main(String[] args) throws Exception{

        final int port = 8888;

        // 相当于 ServerSocket
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        // 绑定端口
        serverSocketChannel.socket().bind(new InetSocketAddress(port));

        // 设置为非阻塞
        serverSocketChannel.configureBlocking(false);

        // 获得selector
        Selector selector = Selector.open();

        // serverSocketChannel 注册到selector当中 OP_ACCEPT连接事件
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        // 循环等待连接处理
        while (true){

            if (0 == selector.select(1000)){
                System.out.println("等待连接....");
                continue;
            }
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            // 获取到所有的注册key
            Iterator<SelectionKey> selectionKeyIterator = selectionKeys.iterator();

            if (selectionKeyIterator.hasNext()) {
                SelectionKey selectionKey = selectionKeyIterator.next();

                // 连接事件处理
                if (selectionKey.isAcceptable()) {
                    // 获得通道
                    SocketChannel accept = serverSocketChannel.accept();
                    // 设置非阻塞
                    accept.configureBlocking(false);
                    // 注册到selector中，并设置为读事件
                    accept.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));

                    System.out.println("客户端连接成功....生成socketChannel" + accept.hashCode());
                }

                // 读事件处理
                if (selectionKey.isReadable()) {


                    SocketChannel channel = (SocketChannel)selectionKey.channel();
                    System.out.println("读事件发生....."+channel.hashCode());
                    ByteBuffer buffer = (ByteBuffer) selectionKey.attachment();
                    channel.read(buffer);

                    System.out.println("读取到客户端到数据...."+new String(buffer.array()));
                }
                // key移除操作
                selectionKeyIterator.remove();
            }

        }



    }
}
```



**客户端**

```java
public class NIOClient {

    public static void main(String[] args) throws Exception{
        // 获得一个Channel
        SocketChannel socketChannel = SocketChannel.open();

        if (!socketChannel.connect(new InetSocketAddress("127.0.0.1",8888))) {
            System.out.println("连接中.....");
        }

        if (!socketChannel.finishConnect()){
            System.out.println("等待连接...");
        }

        ByteBuffer byteBuffer = ByteBuffer.wrap("hello world....".getBytes());

        socketChannel.write(byteBuffer);

        System.out.println("客户端发送完成...");

    }
}
    
```





#### Demo1 - 缓冲区读写

```java
  // 输入流获取管道
        FileInputStream fileInputStream = new FileInputStream(new File("/Users/liuzihao/Downloads/niofile.txt"));
        FileChannel channel = fileInputStream.getChannel();
        
        // 输出流获取管道
        FileOutputStream fileOutputStream = new FileOutputStream(new File("/Users/liuzihao/Downloads/niofile1.txt"));
        FileChannel outChannel = fileOutputStream.getChannel();
        
        // 缓冲区
        ByteBuffer allocate = ByteBuffer.allocate(1);
        
        while (true) {
            // 清除标记位置
            allocate.clear();
            // 将输入流通道里数据 读取到缓冲区
            int read = channel.read(allocate);
            // 数据读取完 跳出
            if (read<= -1){
                break;
            }
            // 转换读写模式
            allocate.flip();
            // 写到输出通道中
            outChannel.write(allocate);
        }
        
        // 关闭流操作
        fileInputStream.close();
        fileOutputStream.close();
```

