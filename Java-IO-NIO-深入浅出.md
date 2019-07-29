1. Tutorial

    Java NIO (New IO)是Java的另一种IO API(源自Java 1.4)，这意味着可以替代标准Java IO和Java网络API。Java NIO提供了一种不同于标准IO API的处理IO的方法。

    * Java NIO: Channels and Buffers
    在标准的IO API中，可以使用字节流和字符流。在NIO中，您使用通道和缓冲区。数据总是从通道读入缓冲区，或者从缓冲区写入通道。

    * Java NIO: Non-blocking IO
    Java NIO使您能够执行非阻塞IO。例如，线程可以请求通道将数据读入缓冲区。当通道将数据读入缓冲区时，线程可以执行其他操作。一旦数据读入缓冲区，线程就可以继续处理它。将数据写入通道也是如此。

    * Java NIO: Selectors
    Java NIO包含“selectors”的概念。selector是一个对象，它可以监视多个通道的事件(如:打开的连接、到达的数据等)。因此，一个线程可以监视多个通道的数据。

    在本系列的下一篇文章——Java NIO概述中，将更详细地解释所有这些工作原理。

2. Overview

    Java NIO由以下核心组件构成:
    
    * Channels
    * Buffers
    * Selectors

    Java NIO有更多的类和组件，但是在我看来， Channel, Buffer和Selector构成了API的核心。其他组件，如Pipe和FileLock，只是与三个核心组件一起使用的实用程序类。因此，我将在NIO概述中重点介绍这三个组件。其他组件将在本教程的其他部分以它们自己的文本解释。请看本页顶部的菜单。

    1. Channels 和 Buffers

        通常，NIO中的所有IO都从一个通道开始。通道有点像流。从通道数据可以读入缓冲区。数据也可以从缓冲区写入通道。下面是一个例子:

        ![](java/java-io-nio-overview-channels-buffers.png)

        Java NIO:通道将数据读入缓冲区，缓冲区将数据写入通道

        有几种Channel和Buffer类型。以下是Java NIO中主要通道实现的列表:

        * FileChannel
        * DatagramChannel
        * SocketChannel
        * ServerSocketChannel

        可以看到，这些通道包括UDP + TCP网络IO和文件IO。

        这些类还附带了一些有趣的接口，但是为了简单起见，我将不介绍这些接口。它们将在本Java NIO教程的其他文本中进行相关解释。

        以下是Java NIO的核心Buffer实现列表:

        * ByteBuffer
        * CharBuffer
        * DoubleBuffer
        * FloatBuffer
        * IntBuffer
        * LongBuffer
        * ShortBuffer

        这些缓冲区涵盖了可以通过IO发送的基本数据类型:byte, short, int, long, float, double和characters。
        Java NIO还有一个MappedByteBuffer，它与内存映射文件一起使用。不过，我将把这个缓冲区排除在概述之外。

    2. Selectors

        选择器允许一个线程处理多个通道。如果您的应用程序打开了许多连接(通道)，但每个连接上的流量都很低，那么这将非常方便。例如，在聊天服务器中。

        下面是一个线程使用选择器处理3个通道的例子:

        ![](java/java-io-nio-overview-selectors.png)

        Java NIO:一个线程使用一个选择器来处理3个通道

        要使用选择器，您需要向它注册通道。然后调用它的select()方法。此方法将阻塞，直到为已注册的通道之一准备好事件为止。一旦方法返回，线程就可以处理这些事件。事件的例子有传入连接、接收的数据等。

3. Channel

    Java NIO通道与流类似，但有一些区别:
    * 您可以对通道进行读写。流通常是单向的(读或写)。
    * 通道可以异步读取和写入。
    * 通道总是读写缓冲区。
    
    如上所述，将数据从通道读入缓冲区，并将缓冲区中的数据写入通道。下面是一个例子:
    ![](java/java-io-nio-overview-channels-buffers.png)

    Java NIO:通道将数据读入缓冲区，缓冲区将数据写入通道
    
    1. 通道的实现

        以下是Java NIO中最重要的通道实现:

        * FileChannel
        * DatagramChannel
        * SocketChannel
        * ServerSocketChannel

        FileChannel从文件中读取数据并将数据读入文件。

        DatagramChannel可以通过UDP在网络上读写数据。

        SocketChannel可以通过TCP在网络上读写数据。

        ServerSocketChannel允许您侦听传入的TCP连接，就像web服务器一样。为每个传入连接创建一个套接字通道。

    2. 基本的Channel例子

        下面是一个使用FileChannel将一些数据读入缓冲区的基本例子:

        ```java
        RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
        FileChannel inChannel = aFile.getChannel();

        ByteBuffer buf = ByteBuffer.allocate(48);

        int bytesRead = inChannel.read(buf);
        while (bytesRead != -1) {

            System.out.println("Read " + bytesRead);
            buf.flip();

            while(buf.hasRemaining()){
                System.out.print((char) buf.get());
            }

            buf.clear();
            bytesRead = inChannel.read(buf);
        }
        aFile.close();
        ```

        注意buf.flip()调用。首先读入缓冲区，然后把它翻转过来，然后你读出来。我将在下一篇关于Buffer的文章中详细介绍。

4. Buffer

    在与NIO通道交互时使用Java NIO Buffers 。如您所知，数据从通道读入缓冲区，然后从缓冲区写入通道。

    缓冲区本质上是一块内存，您可以将数据写入其中，然后再读取数据。这个内存块被封装在NIO Buffer对象中，该对象提供了一组方法，使使用内存块变得更容易。

    * 基于Buffer的使用

        使用缓冲区读取和写入数据通常遵循以下4个步骤:

        1. 将数据写入Buffer
        2. 调用buffer.flip ()
        3. 从Buffer中读取数据
        4. 调用buffer.clear()或buffer.compact()

        当您将数据写入缓冲区时，缓冲区将跟踪您已经写入了多少数据。一旦需要读取数据，就需要使用flip()方法调用将缓冲区从写模式切换到读模式。在读取模式中，缓冲区允许您读取写入缓冲区的所有数据。

        读取完所有数据后，需要清除缓冲区，以便再次编写。有两种方法可以做到这一点:调用clear()或调用compact()。
        * clear方法的作用是:清除整个缓冲区。
        * compact()方法只清除您已经读取的数据。任何未读数据都被移动到缓冲区的开头，现在数据将在未读数据之后写入缓冲区。

        下面是一个简单的缓冲区使用示例，其中的write, flip, read和clear操作以粗体显示:

        ```java
        RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
        FileChannel inChannel = aFile.getChannel();

        //创建容量为48字节的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(48);

        int bytesRead = inChannel.read(buf); //读入缓冲区。

        while (bytesRead != -1) {

            buf.flip();  //准备好读取缓冲区

            while(buf.hasRemaining()){
                System.out.print((char) buf.get()); // 每次读取一个字节
            }

            buf.clear(); //准备好写缓冲区
            bytesRead = inChannel.read(buf);
        }
        aFile.close();
        ```

    * Buffer Capacity, Position 和 Limit

        缓冲区本质上是一块内存，您可以将数据写入其中，然后再读取数据。这个内存块被封装在NIO Buffer对象中，该对象提供了一组方法，使使用内存块变得更容易。
        为了理解缓冲区的工作原理，您需要熟悉缓冲区的三个属性。这些都是:

        * capacity
        * position
        * limit

        position和limit的含义取决于缓冲区是处于读模式还是写模式。无论缓冲模式如何，容量总是相同的。
        这是读写模式下容量、位置和限制的说明。解释在插图之后的部分。

        ![](java/java-io-nio-buffers-modes.png)

        * Capacity

            缓冲区作为内存块，具有一定的固定大小，也称为“容量”。您只能将设定容量的字节、长、字符等写入缓冲区。一旦缓冲区满了，就需要清空它(读取数据或清除数据)，然后才能将更多数据写入缓冲区。

        * Position

            当您将数据写入缓冲区时，您是在特定的位置这样做的。初始位置是0。当一个字节、long等被写入缓冲区时，该位置被提前指向缓冲区中的下一个单元格，以便将数据插入其中。位置可以最大限度地变成容量- 1。

            当从缓冲区读取数据时，也要从给定位置读取数据。当您将缓冲区从写模式切换到读模式时，位置将重置为0。当您从缓冲区读取数据时，您从position读取数据，position将被提前到下一个要读取的位置。

        * Limit

            在写模式下，缓冲区的限制是可以写入缓冲区的数据量的限制。在写模式下，极限等于缓冲区的容量。
            
            当将缓冲区翻转到读取模式时，limit表示您可以从数据中读取多少数据的限制。**因此，当将缓冲区翻转到读取模式时，将limit设置为写入模式的写入位置**。换句话说，您可以读取写入的字节数(限制设置为写入的字节数，由位置标记)。

    * Buffer类型

        Java NIO附带以下缓冲区类型:
        
        * ByteBuffer
        * MappedByteBuffer
        * CharBuffer
        * DoubleBuffer
        * FloatBuffer
        * IntBuffer
        * LongBuffer
        * ShortBuffer

        如您所见，这些缓冲区类型表示不同的数据类型。换句话说，它们让您以char、short、int、long、float或double的形式处理缓冲区中的字节。

        MappedByteBuffer有点特殊，将在它自己的文本中介绍。

    * 分配一个Buffer

        要获得缓冲区对象，必须首先分配它。每个缓冲区类都有一个分配()方法来做这件事。下面是一个例子，展示了一个字节缓冲区的分配，其容量为48字节:

        ```java
        ByteBuffer buf = ByteBuffer.allocation (48);
        ```

        下面是一个分配空间为1024个字符的CharBuffer的例子:

        ```java
        CharBuffer buf = CharBuffer.assign (1024);
        ```

    * 往Buffer写数据

        你可以用两种方式将数据写入缓冲区:
        * 将数据从Channel写入缓冲区
        * Buffer自己通过put()方法将数据写入缓冲区。

        下面是一个例子，展示了一个通道如何将数据写入缓冲区:
        
        ```java
        int bytesRead = inChannel.read(buf);/ /读取到缓冲区
        ```

        下面是一个通过put()方法将数据写入缓冲区的例子:

        ```java
        buf.put (127);
        ```

        put()方法还有许多其他版本，允许以多种不同的方式将数据写入缓冲区。例如，在特定位置写入，或将字节数组写入缓冲区。有关具体缓冲区实现的详细信息，请参阅JavaDoc。

    * flip()

        flip()方法将缓冲区从写模式切换到读模式。调用flip()将position设置回0，并设置limit为position的位置。
        
        换句话说，position现在标记读取位置，limit标记写入缓冲区的字节数、字符数等—可以读取的字节数、字符数等的限制。

    * 从Buffer读数据

        从缓冲区读取数据有两种方法：
        * 将数据从缓冲区读入Channel。
        * Buffer自己使用get()方法从缓冲区读取数据。

        下面是一个如何从缓冲区读取数据到通道的例子:
        
        ```java
        int byteswriting = inChannel.write(buf);//从缓冲区读入通道
        ```

        下面是一个使用get()方法从缓冲区读取数据的例子:

        ```java
        byte aByte = buf.get();
        ```

        get()方法还有许多其他版本，允许您以多种不同的方式从缓冲区读取数据。例如，读取特定位置，或从缓冲区读取字节数组。有关具体缓冲区实现的详细信息，请参阅JavaDoc。

    * rewind()

        rewind()将position设置回0，这样就可以重新读取缓冲区中的所有数据。这个限制保持不变，因此仍然标记可以从缓冲区读取多少元素(字节、字符等)。

    * clear() 和 compact()

        一旦完成了从缓冲区中读取数据，就必须使缓冲区为再次写入做好准备。您可以通过调用clear()或调用compact()来实现这一点。

        **如果调用clear()，则positioni将被设置为0和limit设置为capacity**。换句话说，缓冲区被清除。缓冲区中的数据未被清除。只有指示可以将数据写入缓冲区的位置的标记。

        如果在调用clear()时缓冲区中有任何未读数据，则该数据将被“遗忘”，这意味着不再有任何标记来表示哪些数据已被读取，哪些数据尚未被读取。

        如果缓冲区中仍然有未读数据，并且您希望稍后读取它，但是需要先进行一些写入操作，那么调用compact()而不是clear()。

        **compact()将所有未读数据复制到缓冲区的开头。然后将position设置为最后一个未读元素之后的正确位置**。与clear()一样，limit属性仍然被设置为capacity。现在缓冲区已经准备好进行写入，但是不会覆盖未读数据。

    * mark() 和 reset()

        您可以通过调用Buffer.mark()方法来标记缓冲区中的给定位置。然后，您可以通过调用Buffer.reset()方法将postion重置回标记的位置。举个例子:

        ```java
        buffer.mark();

        //call buffer.get() a couple of times, e.g. during parsing.

        buffer.reset();  //将位置设置回标记。
        ```

    * equals() 和 compareTo()

        可以使用equals()和compareTo()比较两个缓冲区。

        * equals()

            两个缓冲区是相等的，需要满足以下条件:
            
            * 它们具有相同的类型(字节、字符、int等)
            * 它们在缓冲区中有相同数量的剩余字节、字符等。
            * 所有剩余的字节、字符等都是相等的。

            如您所见，equals只比较缓冲区的一部分，而不是其中的每个元素。实际上，它只是比较缓冲区中的其余元素。

        * compareTo()

            compareTo()方法比较两个缓冲区中剩余的元素(字节、字符等)，用于例如排序例程。一个缓冲区被认为比另一个缓冲区“更小”，需要满足以下条件:

            * 第一个元素等于另一个缓冲区中对应的元素，小于另一个缓冲区中的元素。
            * 所有的元素都是相等的，但是第一个缓冲区会在第二个缓冲区之前耗尽元素(它的元素更少)。

5. Scatter / Gather

    Java NIO提供了内置的scatter/gather支持。scatter/gather是用于从通道中读取和从通道中写入的概念。

    从通道散射读取是将数据读入多个缓冲区的读取操作。因此，通道“散射”将数据从通道“分散”到多个缓冲区。

    对通道的收集写操作是将多个缓冲区中的数据写入到单个通道的写操作。因此，通道“收集”来自多个缓冲区的数据到一个通道中。

    scatter/gather在需要单独处理传输数据的各个部分的情况下非常有用。例如，如果消息由头和正文组成，则可以将头和正文保存在单独的缓冲区中。这样做可以使您更容易地分别处理header和body。

    1. Scattering读取

        “分散读取”将数据从单个通道读入多个缓冲区。下面是这一原则的一个例子:

        下面是散点原理的一个例子:

        ![](java/java-io-nio-scatter.png)

        下面是一个代码示例，演示了如何执行散射读取:

        ```java
        ByteBuffer header = ByteBuffer.allocate(128);
        ByteBuffer body   = ByteBuffer.allocate(1024);

        ByteBuffer[] bufferArray = { header, body };

        channel.read(bufferArray);
        ```

        注意，首先将缓冲区插入数组，然后将数组作为参数传递给channel.read()方法。然后read()方法按照缓冲区在数组中出现的顺序从通道中写入数据。一旦缓冲区满了，通道将继续填充下一个缓冲区。

        分散读取在进入下一个缓冲区之前会填满一个缓冲区，这意味着它不适合动态调整消息部分的大小。换句话说，如果您有一个头和一个主体，并且头的大小是固定的(例如128字节)，那么分散读取就可以正常工作。

    2. Gathering写入

        “收集写”将数据从多个缓冲区写到一个通道。下面是这一原则的一个例子:

        ![](java/java-io-nio-gather.png)

        下面是一个代码示例，展示了如何执行一个集合写:

        ```java
        ByteBuffer header = ByteBuffer.allocate(128);
        ByteBuffer body   = ByteBuffer.allocate(1024);

        //write data into buffers

        ByteBuffer[] bufferArray = { header, body };

        channel.write(bufferArray);
        ```

        缓冲区数组被传递到write()方法中，该方法按数组中遇到的顺序写入缓冲区的内容。只写入缓冲区的位置和限制之间的数据。因此，如果一个缓冲区的容量为128字节，但只包含58字节，则只有58字节从该缓冲区写到通道。**因此，与分散读取相比，聚集写入可以很好地处理动态大小的消息部分。**

6. Channel to Channel Transfers

    在Java NIO中，如果其中一个通道是FileChannel，则可以直接将数据从一个通道传输到另一个通道。FileChannel类有一个transferTo()和一个transferFrom()方法，它们可以为您完成这一任务。

    1. transferFrom()

        FileChannel.transferfrom()方法将数据从源通道传输到FileChannel。下面是一个简单的例子:

        ```java
        RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
        FileChannel      fromChannel = fromFile.getChannel();

        RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
        FileChannel      toChannel = toFile.getChannel();

        long position = 0;
        long count    = fromChannel.size();

        toChannel.transferFrom(fromChannel, position, count);
        ```

        参数position和count，告诉目标文件中从何处开始写入(position)，以及最大传输多少字节(count)。如果源通道的字节数少于count，那么传输的字节数就会减少。
        
        此外，一些SocketChannel实现可能只传输SocketChannel在其内部缓冲区中已经准备好的数据——即使SocketChannel稍后可能有更多可用数据。因此，它可能不会将请求的全部数据(count)从SocketChannel传输到FileChannel。

    2. transferTo()

        transferTo()方法从一个FileChannel传输到另一个通道。下面是一个简单的例子:

        ```java
        RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
        FileChannel      fromChannel = fromFile.getChannel();

        RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
        FileChannel      toChannel = toFile.getChannel();

        long position = 0;
        long count    = fromChannel.size();

        fromChannel.transferTo(position, count, toChannel);
        ```

        请注意这个示例与前面的示例有多么相似。唯一真正的区别是方法调用的是哪个FileChannel对象。其余的都是一样的。

        transferTo()方法也存在SocketChannel的问题。SocketChannel实现只能从FileChannel传输字节，直到发送缓冲区被填满，然后停止。

7. Selector

    Java NIO Selector是一个组件，它可以检查一个或多个Java NIO通道实例，并确定哪些通道已经准备好，例如读取或写入。通过这种方式，一个线程可以管理多个通道，从而管理多个网络连接。

    * 为什么使用Selector?

        只使用一个线程来处理多个通道的好处是，您需要更少的线程来处理通道。实际上，您可以只使用一个线程来处理所有通道。对于操作系统来说，在线程之间切换非常昂贵，而且每个线程也会占用操作系统中的一些资源(内存)。因此，使用的线程越少越好。

        但是请记住，现代操作系统和CPU在多任务处理方面变得越来越好，因此多线程的开销会随着时间的推移变得越来越小。事实上，如果一个CPU有多个内核，那么不进行多任务处理可能会浪费CPU的能量。无论如何，关于设计的讨论属于别的范畴。这里只需说明，您可以使用选择器使用一个线程处理多个通道。

        下面是一个线程使用选择器处理3个通道的例子:

        ![](java/java-io-nio-overview-selectors.png)

    * 创建Selector

        您可以通过调用Select.open()方法来创建选择器，如下所示:

        ```java
        Selector selector = Selector.open();
        ```

    * 注册Channels到Selector

        要使用带有选择器的通道，必须用选择器注册通道。这是使用SelectableChannel.register()方法完成的，如下所示:

        ```java
        channel.configureBlocking(false);

        SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
        ```

        **通道必须处于非阻塞模式，才能与选择器一起使用**。这意味着您不能将FileChannel与选择器一起使用，因为FileChannel不能切换到非阻塞模式。不过套接字通道可以很好地工作。

        注意register()方法的第二个参数。这是一个“兴趣集”，意思是通过选择器在通道中侦听您感兴趣的事件。你可以收听四个不同的事件:
        
        1. Connect
        2. Accept
        3. Read
        4. Write
        
        一个“触发事件”的通道也被称为为该事件“准备好了”。

        * 成功连接到另一个服务器的通道是“connect ready”。
        * 接受传入连接的服务器套接字通道是“accept”就绪的。
        * 准备读取数据的通道是“read”ready。
        * 准备好向其写入数据的通道是“write”ready。

        这四个事件由四个SelectionKey常量表示:
        
        1. SelectionKey.OP_CONNECT
        2. SelectionKey.OP_ACCEPT
        3. SelectionKey.OP_READ
        4. SelectionKey.OP_WRITE
        
        如果你对不止一个事件感兴趣，或者把常数放在一起，就像这样:

        ```java
        int interestSet = SelectionKey。OP_READ | SelectionKey.OP_WRITE;
        ```

        我将回到这篇文章后面的兴趣点。

    * SelectionKey

        正如您在前一节中看到的，当您使用选择器注册通道时，register()方法将返回SelectionKey对象。这个SelectionKey对象包含一些有趣的属性:

        * interest set
        * ready set
        * Channel
        * Selector
        * attached object (optional)

        我将在下面描述这些属性。

        1. Interest Set

            兴趣集是您对“选择”感兴趣的事件集，如“向选择器注册通道”一节所述。你可以像这样通过SelectionKey读写兴趣集:

            ```java
            int interestSet = selectionKey.interestOps();

            boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
            boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
            boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
            boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;   
            ```

        2. Ready Set

            就绪集是通道准备进行的一组操作。您将主要在选择之后访问就绪集。选择将在后面的部分进行解释。您可以这样访问就绪集:

            ```java
            int readySet = selectionKey.readyOps();
            ```

            您可以使用与兴趣集相同的方法测试通道准备用于哪些事件/操作。但是，你也可以用这四种方法来代替，它们都是布尔值:

            ```java
            selectionKey.isAcceptable();
            selectionKey.isConnectable();
            selectionKey.isReadable();
            selectionKey.isWritable();
            ```
            
        3. Channel + Selector

            从SelectionKey访问channel + Selector非常简单。具体做法如下:

            ```java
            Channel  channel  = selectionKey.channel();

            Selector selector = selectionKey.selector();
            ```

        4. Attaching Objects

            您可以将对象附加到SelectionKey上——这是识别给定通道或将进一步信息附加到通道上的方便方法。例如，可以将正在使用的缓冲区附加到通道中，或者附加一个包含更多聚合数据的对象。下面是如何附加对象:

            ```java
            selectionKey.attach(theObject);

            Object attachedObj = selectionKey.attachment();
            ```

            还可以在register()方法中用选择器注册通道时附加一个对象。它是这样的:

            ```java
            SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
            ```

    * 通过Selector选择通道

        一旦用选择器注册了一个或多个通道，就可以调用select()方法之一。这些方法返回为您感兴趣的事件(连接、接受、读取或写入)“准备好”的通道。换句话说，如果您对准备读取的通道感兴趣，那么您将从select()方法接收准备读取的通道。
        以下是select()方法:

        * int select()
        * int select(long timeout)
        * int selectNow()

        1. select()阻塞，直到至少有一个通道为您注册的事件准备好。
        2. select(long timeout)与select()执行相同的操作，只是它阻塞超时毫秒(参数)的最大值。
        3. selectNow()根本不会阻塞。它立即返回任何准备好的通道。

        select()方法返回的int值告诉我们有多少通道已经准备好了。也就是说，自上次调用select()以来，已经准备好了多少通道。如果您调用select()，它返回1，因为一个通道已经就绪，您再次调用select()，并且一个通道已经就绪，它将再次返回1。如果您没有对第一个就绪的通道执行任何操作，那么现在就有了两个就绪通道，但是在每个select()调用之间只有一个通道已经就绪。

        **selectedKeys()**

        一旦您调用了select()方法中的一个，并且它的返回值表明一个或多个通道已经就绪，您就可以通过“selected key set”通过调用selectors selectedKeys()方法来访问就绪通道。它是这样的:

        ```java
        Set<SelectionKey> selectedKeys = select .selectedKeys();
        ```

        当您使用选择器注册通道时，channel.register()方法将返回SelectionKey对象。此键表示使用该选择器进行通道注册。您可以通过selectedKeySet()方法访问这些密钥。从SelectionKey。
        您可以迭代这个选定的键集来访问就绪通道。它是这样的:

    * wakeUp()
    * close()
    * 完整的Selector例子

8. FileChannel
9. SocketChannel
10. ServerSocketChannel
11. Non-blocking Server
12. DatagramChannel
13. Pipe
14. NIO　vs. IO
15. Path
16. Files
17. AsynchronousFileChannel




原文：http://tutorials.jenkov.com/java-nio/index.html