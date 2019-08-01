## netty overview

1. 核心组件

    核心组件包括：
    * Channels
    * Callbacks
    * Futures
    * Events and handlers

    1. Channels

        Channel是Java NIO的基本构成

    2. Callbacks

        回调只是一个方法，它的引用已经提供给另一个方法。这使得后者能够在适当的时候调用前者。

        Netty在处理事件时在内部使用回调;当回调被触发时，事件可以由接口ChannelHandler的实现来处理。

        下一个清单显示了一个示例:当建立了一个新连接时调用ChannelHandler回调channelActive()并打印一条消息。

        ```java
        public class ConnectHandler extends ChannelInboundHandlerAdapter {
            @Override
            public void channelActive(ChannelHandlerContext ctx)
                throws Exception {                                      
                System.out.println(
                    "Client " + ctx.channel().remoteAddress() + " connected");
            }
        }
        ```

    3. Futures

        Future提供了在操作完成时通知应用程序的另一种方法。此对象充当异步操作结果的占位符;它将在将来的某个时候完成，并提供对结果的访问。

        JDK附带java.util.concu rrent.Future接口，但所提供的实现只允许手动检查操作是否已完成或阻塞，直到完成为止。这非常麻烦，因此Netty提供了自己的实现ChannelFuture，以便在执行异步us操作时使用。

        ChannelFuture提供了额外的方法，允许我们注册一个或多个ChannelFutureListener实例。侦听器的回调方法operationComplete()在操作完成时调用。然后，侦听器可以确定操作是否成功完成或是否有错误。如果是后者，则可以检索发生的Throwable。简而言之，ChannelFutureListener提供的通知机制消除了手动检查操作完成情况的需要

    4. Events and handlers

        Netty使用不同的事件来通知我们操作的状态或状态的变化。这允许我们根据已发生的事件触发适当的操作。这些操作可能包括

        * 记录日志
        * 数据转换
        * 流控制
        * 应用逻辑

        Netty是一个网络框架，因此事件按照它们与入站或出站数据流的相关性进行分类。可能由入站数据或关联的状态更改触发的事件包括

        * 活动连接或非活动连接
        * 数据读取
        * 用户事件
        * 错误事件

        出站事件是将来可能触发某个动作的操作的结果

        * 打开或关闭到远程对等点的连接
        * 将数据写入或刷新到套接字

        ![](netty/netty-overview-core-component.png)

2. Netty第一个应用

    1. server端

        ```java
        @Sharable  //表明一个ChannelHandler可以被多个通道安全地共享
        public class EchoServerHandler extends ChannelInboundHandlerAdapter {

            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) {
                ByteBuf in = (ByteBuf) msg;
                System.out.println("Server received: " + in.toString(CharsetUtil.UTF_8)); //将消息记录到控制台
                ctx.write(in); //将接收到的消息写入发送方，而不刷新出站消息
            }

            @Override
            public void channelReadComplete(ChannelHandlerContext ctx) {
                ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE); //将挂起的消息刷新到远程对等端并关闭通道
            }

            @Override
            public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
                cause.printStackTrace(); //打印异常堆栈跟踪
                ctx.close(); //关闭通道
            }
        }
        ```

        ```java
        public class EchoServer {
            private final int port;

            public EchoServer(int port) {
                this.port = port;
            }

            public static void main(String[] args) throws Exception {
                if (args.length != 1) {
                    System.err.println("Usage: " + EchoServer.class.getSimpleName() + " <port>");
                }
                int port = Integer.parseInt(args[0]);
                new EchoServer(port).start();
            }

            public void start() throws Exception {
                final EchoServerHandler serverHandler = new EchoServerHandler();
                EventLoopGroup group = new NioEventLoopGroup(); //创建EventLoopGroup
                try {
                    ServerBootstrap b = new ServerBootstrap(); //创建ServerBootstrap
                    b.group(group)
                    .channel(NioServerSocketChannel.class) //指定NIO传输的使用Channel
                    .localAddress(new InetSocketAddress(port)) //使用指定的端口设置套接字地址
                    .childHandler(new ChannelInitializer<SocketChannel>() { //增加了一个EchoServerHandler到通道的ChannelPipeline
                        @Override
                        public void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(serverHandler);
                        }
                    });
                    ChannelFuture f = b.bind().sync(); //异步绑定服务器;sync()等待绑定完成。
                    f.channel().closeFuture().sync(); //获取通道的closeFuture并阻塞当前线程，直到它完成为止
                } finally {
                    group.shutdownGracefully().sync(); //关闭EventLoopGroup，释放所有资源
                }
            }
        }
        ```

        总结：
        1. EchoServerHandler实现了业务逻辑
        2. main方法启动了server

        启动中涉及的步骤：
        1. 创建ServerBootstrap实例并绑定server
        2. 创建并分配NioEventLoopGroup实例来处理事件processing，例如接受新连接和读取/写入数据。
        3. 指定服务器绑定到的本地InetSocketAddress。
        4. 使用EchoServerHandler实例初始化每个新通道。
        5. 调用ServerBootstrap.bind()绑定服务器。

    2. client端

        client端步骤：
        1. 连接到服务器
        2. 发送一条或多条消息
        3. 对于每个消息，等待并接收来自服务器的相同消息
        4. 关闭连接

        ```java
        @Sharable // 将该类标记为实例可在通道之间共享的类
        public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {
            @Override
            public void channelActive(ChannelHandlerContext ctx) { // 当通知通道处于活动状态时，发送一条消息
                ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!", CharsetUtil.UTF_8));
            }

            @Override
            public void channelRead0(ChannelHandlerContext ctx, ByteBuf in) { // 记录接收到的消息的转储
                System.out.println("Client received: " + in.toString(CharsetUtil.UTF_8));
            }

            @Override
            public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // 在异常情况下，记录错误并关闭通道
                cause.printStackTrace();
                ctx.close();
            }
        }
        ```

        ```java
        public class EchoClient {
            private final String host;
            private final int port;

            public EchoClient(String host, int port) { //设置服务器的InetSocketAddress
                this.host = host;
                this.port = port;
            }
            
            public static void main(String[] args) throws Exception {
                if (args.length != 2) {
                    System.err.println("Usage: " + EchoClient.class.getSimpleName() + " <host> <port>");
                    return;
                }
                String host = args[0];
                int port = Integer.parseInt(args[1]);
                new EchoClient(host, port).start();
            }
            
            public void start() throws Exception {
                EventLoopGroup group = new NioEventLoopGroup();
                try {
                    Bootstrap b = new Bootstrap();//创建Bootstrap
                    b.group(group)	//指定处理客户端事件的EventLoopGroup；需要实现NIO。
                    .channel(NioSocketChannel.class) //通道类型是NIO传输的类型
                    .remoteAddress(new InetSocketAddress(host, port))
                    .handler(new ChannelInitializer<SocketChannel>() {	//在创建通道时向管道添加EchoClientHandler
                        @Override
                        public void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new EchoClientHandler());
                        }
                    });
                    ChannelFuture f = b.connect().sync();
                    f.channel().closeFuture().sync();
                } finally {
                    group.shutdownGracefully().sync();
                }
            }
        }
        ```