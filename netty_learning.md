### netty 笔记

[TOC]

#### 一、服务启动

##### （1）主线

* our thread

  1. 创建selector
  2. 创建 server socket channel
  3. 初始化server socket channel
  4. 给server socket channel 从boss group中选择一个NioEventLoop

* Boss thread

  1. 将server socket channel 注册到选择的NioEventLoop的selector
  2. 绑定地址启动
  3. 注册接受连接事件(OP_ACCEPT)到selector上

##### （2）知识点

* 启动服务的本质：

  ```java
  Selector selector = sun.nio.ch.SelectorProviderImpl.openSelector();
  ServerSocketChannel serverSocketChannel = provider.openServerSocketChannel();
  selectionKey = javaChannel().register(eventLoop().unwrappedSelector(),0,this);
  javaChannel().bind(localAddress,config.getBacklog());
  selectionKey.interestOps(OP_ACCEPT);
  ```

* Selector是在new NioEventLoopGroup()(创建一批NioEventLoop)时创建

* 第一次register是并不是监听OP_ACCEPT,而是0

  ```java
  selectionKey = javaChannel().register(eventLoop().unwrappedSelector(),0,this);
  ```

* 最终监听OP_ACCEPT是通过bind完成后的fireChannelActive()来触发的

* NioEventLoop是通过Register操作的执行来完成启动的

* 类似Channelnitializer,一些Handler可以设计成一次性的，用完就移除，例如授权



#### 二、构建连接

##### （1）主线

* boos thread
  * NioEventLoop中的selector轮询创建连接事件（OP_ACCEPT）
  * 创建socket channel
  * 初始化socket channel 并从<span style="color:red">worker</span> group中选择一个NioEventLoop
* Worker thread
  * 将socket channel 注册到选择的NioEventLoop的selector
  * 注册读事件（OP_READ）到selector上

##### （2）知识点

* 接受连接本质’：

  selector.select()/selectNow()/select(timeoutMills)发现
  OP_ACCEPT事件，处理：

  ```java
  SocketChannel socketChannel = serverSocketChannel.accept();
  selectKey = javaChannel().register(eventLoop().unwrappedSelector(),0,this);
  selectKey.interestOps(OP_READ);
  ```

* 创建连接的初始化和注册是通过pipeline.fireChannelRead在ServerBootstrapAcceptor中完成的

* 第一次Register并不是监听OP_READ，而是0

  ```java
  selectKey = javaChannel().register(eventLoop().unwrappedSelector(),0,this);
  ```

* 最终监听OP_READ是通过“Register”完成后的fireChannelActive(io.netty.channel.AbstractChannel.AbstractUnsafe#register()中)来触发的

* Worker's NIOEventLoop是通过Register操作执行来启动

* 接受连接的读操作，不会尝试读取更多次（16次）

#### 三、接收数据

##### （1）读数据技巧

1. 自适应数据大小的分配器（AdaptiveRecvByteBufAllocator）
2. 连续读（defaultMaxMessagesPerRead）

##### （2）主线

* 多路复用器（Selector）接收到OP_READ事件
* 处理OP_READ事件：NioSocketChannel.NioSocketChannelUnsafe.read()
  * 分配一个初始1024字节的byte buffer来接受数据
  * 从Channel接受数据到byte buffer
  * 记录实际接受数据大小，调整下次分配byte buffer大小
  * 触发 pipeline.fireChannelRead(byteBuf)把读取到的数据传播出去
  * 判断接受byte buffer是否满载而归：是，尝试继续读取直到没有数据或者满16次；否，结束本轮读取，等待下次OP_READ事件

##### （3）知识点

* 读取数据的本质：sun.nio.ch.SocketChannelImpl#read(java.nio.ByteBuffer)

* NioSocketChannel read()是读数据，NioServerSocketChannel read()是创建连接

* Pipeline.fireChannelReadComplete();一次读事件处理完成

  Pipeline.fireChannelRead(byteBuf);一次读数据完成，一次读事件处理可能会包含多次读数据操作

* 为什么最多只尝试读取16次？“雨露均沾”

* AdaptiveRecvByteBufAllocator 对 byteBuf的猜测：放大果断，缩小谨慎（需要连续两次判断）



#### 四、业务处理

##### （1）主线

* 触发 pipeline.fireChannelRead(byteBuf)把读取到的数据传播出去

  Handler执行资格：

  * 实现了ChannelInboundHandler
  * 实现方法channelRead 不能加注解@Skip

##### （2）知识点

* 处理业务的本质：数据在pipeline中所有的handler的channelRead()执行过程

  Handler要实现io.netty.channel.ChannelInboundHandler#channelRead(ChannelHandlerContext ctx,Object msg),且不能加注解@Skip才能被执行到

  中途可退出，不保证执行到Tail Handler

* 默认处理线程就是Channel绑定的NioEventLoop线程，也可以设置其他：pipeline.addLast(new UnorderedThreadPoolEventExecutor(10),serverHandler)



#### 五、发送数据

##### （1）写数据的三种方式

1. Write:写到一个buffer
2. 把buffer你的数据发送出去
3. 写到buffer并立马发送

##### （2）写数据的要点

1. netty写数据，写不进去，会停止写，然后注册一个**OP_WRITE**事件，来通知什么时候可以写进去再写
2. netty批量写数据时，如果想写的都写进去了，接下来的尝试写更多(调整**maxBytesPerGatheringWrite**)
3. netty只要有数据写，且能写的出去，则一直尝试，直到写不出去或者满16次(**writeSpinCount**)
4. netty待写数据太多，超过一定的水位线（**writeBufferWaterMark.high()**），会将可写的标志位改成false,让应用端自己做决定要不要发送数据了

##### （3）主线

![](https://tva1.sinaimg.cn/large/0082zybpgy1gbqh8rdi5ij30uw0u0416.jpg)

* Write-写数据到buffer:

  ChannelOutboundBuffer#addMessage

* Flush-发送buffer里面的数据：

  AbstractChannel-AbstractUnsafe#flush

  * 准备数据：ChannelOutboundBuffer#addFlush
  * 发送：NioSocketChannel#doWrite

##### （4）知识点

* 写的本质：

  * Single write: sun.nio.ch.SocketChannelImpl#write(java.nio.ByteBuffer)
  * Gathering write: sun.nio.ch.SocketChannelImpl#write(java.nio.ByteBuffer[],int,int)

* 写数据写不进去时，会停止写，注册一个<span style="color:red">OP_WRITE</span>事件，来通知什么时候可以写进去了

* <span style="color:red">OP_WRITE不是说有数据可写，而是说可以写进去</span>，所以正常情况，不能注册，否则一直触发

* 批量写数据是，如果尝试写的都写进去了，接下来会尝试写更多（**maxBytesPerGatheringWrite**）

* 只要有数据要写，且能写，则一直尝试，直到16次（**writeSpinCount**）,写16次还没有写完，就直接schedule一个task来继续写，而不是用注册写事件来触发，更简洁有力

* 待写数据太多，超过一定的水位线（writeBufferWaterMark.high()),会将可写的标志位改成false，让应用端子机做决定要不要继续写

* channelHandlerContext.channel().write():从TailContext开始执行

  channelHandlerContext.write:从当前的Context开始



#### 六、断开连接

##### （1）主线

* 多路复用器（Selector）接收到OP_READ事件
* 处理OP_READ事件：NioSocketChannel.NioSocketChannelUnsafe.read()
  * 接收数据
  * 判断接收的数据大小是否<0，如果是，说明是关闭，开始执行关闭：
    * 关闭channel(包含cancel多路复用器的key)
    * 清理消息：不接受新信息，fail掉所有queue中的消息
    * 触发fireChannelInactive和fireCahnnelUnregistered

##### （2）知识点

* 关闭连接的本质：
  * java.nio.channels.spi.AbstractInterruptibleChannel#close
    * java.nio.channels.SelectionKey#cancel
* 要点：
  * 关闭连接，会触发OP_READ方法。读取字节数-1代表关闭
  * 数据读取进行时，强行关闭，触发IOException，进而执行关闭
  * Channel的关闭包含了SelectionKey的cancel



#### 七、关闭服务

##### （1）主线

*  bossGroup.shutdownGracefully();

  workerGroup.shutdownGracefully();

  关闭所有group中的NioEventLoop:

  * 修改NioEventLoop的State标志位
  * NioEventLoop判断State执行退出

  ![](https://tva1.sinaimg.cn/large/0082zybpgy1gbqikk7pupj30wn0u0n1x.jpg)

##### （2）知识点

* 关闭服务本质：

  * 关闭所有连接及Selector：

    * java.nio.channels.Selector#keys

      * java.nio.channels.spi.AbstractInterruptibleChannel#close
      * java.nio.channels.SelectionKey#cancel

    * Selector.close()

  * 关闭所有线程：退出循环体 for(;;)

* 关闭服务要点：

  * 优雅（DEFAULT_SHUTDOWN_QUIT_PERIOD)
  * 可控（DEFAULT_SHUTDOWN_TIMEOUTY)
  * 先不接活，后尽量干完手头的活（先关boss后关worker；不是100%保证）
  



###### 补充：netty性能优化的极致体现在哪些细节上？

1. 通过反射或者Unsafe的方式替换Selector中的selectedKeys的Set实现，采用数组实现的set(默认是HashSet)，充分利用cpu缓存行
2. EventLoop的taskQueue采用MPSC（多个生产者单个消费者）的队列实现，因为eventloop只绑定一个线程，所以可以有优化空间，具体实现细节：采用无锁形式，利用Unsafe的cas能力
3. EventExecutorChooser的选择，如果是2的幂次方，采用位运算，否则取模
4. FastThreadLocal + FastThreadLocalThread + InternalThreadLocalMap, 优化核心点：数组替换Map
5. 不直接使用jdk的SelectorProvider.provider()，而是共享一个provider
6. 通过 冗余long字段，解决缓存行伪共享问题

