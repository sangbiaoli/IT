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
6. Channel to Channel Transfers
7. Selector
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