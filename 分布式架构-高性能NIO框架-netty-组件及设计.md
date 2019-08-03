## netty组件及设计

1. Channel,EventLoop和ChannelFuture

    下面对Channel,EventLoop和ChannelFuture的讨论，它们都是Netty网络的抽象。

    * Channel—Sockets
    * EventLoop—控制流，多线程，高并发
    * ChannelFuture—异步通知

    1. 接口Channel

        基本的I/O操作(bind()、connect()、read()和write()依赖于由底层网络传输提供的基本类型。在基于java的网络中，基本结构是类Socket。Netty的通道接口提供了一个非常好的API
        减少直接使用套接字的复杂性。此外，Channel是具有许多预定义的专门实现的广泛类层次结构的根，其中的一个简短列表如下:

        * EmbeddedChannel
        * LocalServerChannel
        * NioDatagramChannel
        * NioSctpChannel
        * NioSocketChannel

    2. 接口EventLoop

        EventLoop定义了Netty的核心抽象，用于处理一个connection生命周期中发生的事件。图从高层次说明了Channels, EventLoops, Threads和EventLoopGroups之间的关系。

        它们的关系如下：

        * 一个EventLoopGroup包含一个或多个EventLoop
        * EventLoop的生命周期被绑定到单个线程
        * EventLoop处理的所有I/O事件都在其专用线程上处理
        * Channel使用一个EventLoop注册其生命周期
        * 一个EventLoop可以分配给一个或多个Channel

        ![](netty/netty-design-event_loop.png)

        请注意，在这种设计中，给定通道的I/O由相同的I/O执行线程，实际上消除了同步的需要。

    3. 接口ChannelFuture

        正如我们所解释的，Netty中的所有I/O操作都是异步的。因为一个操作可能不会立即返回，所以我们需要一种方法在稍后的时间确定它的结果。

        为此，Netty提供了ChannelFuture，它的addListener()方法注册了一个ChannelFutureListener，以便在操作完成时得到通知(无论成功与否)

2. ChannelHandler和ChannelPipeline

    现在，我们将更详细地了解管理数据流和执行应用程序处理逻辑的组件。

    1. 接口ChannelHandler

        从应用程序开发人员的角度来看，Netty的主要组件是ChannelHandler，它充当所有应用程序逻辑的容器，用于处理入站和出站数据。这是可能的，因为ChannelHandler方法是由网络事件触发的(其中术语“事件”使用得非常广泛)。事实上，ChannelHandler几乎可以用于任何类型的操作，比如将数据从一种格式转换为另一种格式，或者处理处理过程中抛出的异常。

        例如，ChannelInboundHandler是您将经常实现的子接口。此类型接收应用程序的业务逻辑要处理的入站事件和数据。在向连接的客户机发送响应时，还可以刷新ChannelInboundHandler中的数据。应用程序的业务逻辑通常驻留在一个或多个ChannelInboundHandlers中。

    2. 接口ChannelPipline

        ChannelPipeline为ChannelHandlers链提供了一个容器，并定义了一个API，用于沿着该链传播入站和出站事件流。当创建Channel时，它会自动分配自己的ChannelPipeline。

        ChannelPipeline中安装的ChannelHandlers如下:
        * ChannelInitializer实现通过ServerBootstrap注册。
        * 当调用ChannelInitializer.initchannel()时，ChannelInitializer在管道中安装一组自定义的ChannelHandlers。
        * ChannelInitializer将自己从ChannelPipeline中移除。

        让我们更深入地了解ChannelPipeline和ChannelHandler之间的共生关系，以检查在发送或接收数据时数据发生了什么变化。ChannelHandler是专门为支持广泛的用途而设计的，
        您可以将它看作任何处理传入和通过ChannelPipeline的事件(包括数据)的代码的通用容器。

        ![](netty/netty-design-channel_handler.png)

        事件在管道中的移动是在应用程序初始化或引导阶段安装的ChannelHandlers的工作。这些对象接收事件，执行它们所具有的处理逻辑，并将数据传递给链中的下一个处理程序。它们执行的顺序由它们被添加的顺序决定。对于所有实际用途，我们将ChannelHandlers的这种有序排列称为ChannelPipeline。

        ![](netty/netty-design-channel_pipe_line.png)

    3. ChannelHandler详解
    4. Encoders和decoders
    5. 抽象类SimpleChannelInboundHandler

3. Bootstrapping

4. 总结

