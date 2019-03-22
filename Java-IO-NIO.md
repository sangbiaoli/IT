## Java NIO

1. Buffers

    NIO基于Buffer，通过通道方式从I/O服务中发送或接收数据。

    * Buffer属性
        * capacity:总容量
        * limit:标记着首个不能被读或被写的下标
        * position:下一个要读或要写的下标
        * mark:一个初始标记下标，rewind()调用后，position会被重置，mark被置为-1。

        mark <= position <= limit <= capacity
    
    * Buffer及子类
        
        方法|说明
        --|--
        Object array()|返回支持buffer的数组
        int arrayOffset() |返回buffer的数组中首个buffer元素的下标
        int capacity()|返回buffer的容量大小
        Buffer clear()|清空buffer，返回此buffer
        Buffer flip()|设置游标，limit置为position，position置为0，返回此buffer
        boolean hasArray()|此缓冲区有数组组成且不是只读，则返回true，否则返回false
        boolean hasRemaining()|如果至少有一个元素在buffer中，则返回true
        boolean isDirect()|此buffer是直接字节buffer，则返回true
        boolean isReadOnly()|只读则返回true
        int limit()|返回limit
        Buffer limit(int newLimit)|设置limit值
        Buffer mark()|设置mark到position，返回此buffer
        int position()|返回position
        Buffer position(int newPosition|设置position值，返回此buffer
        int remaining()|返回position和limit之间元素数量
        Buffer reset()|重置buffer的position到前一次标记的位置
        Buffer rewind()|跟初始化一样，position置为0，mark为-1

        **Buffer不是线程安全的，所以在多线程环境中需要添加synchronization保持同步**

        Buffer是一个抽象类，java.nio包含了几个抽象类，都是基本类型且继承于Buffer
        * Boolean
        * ByteBuffer
        * CharBuffer
        * DoubleBuffer
        * FloatBuffer
        * IntBuffer
        * LongBuffer
        * ShortBuffer

        ```java
        import java.nio.Buffer;
        import java.nio.ByteBuffer;

        public class BufferDemo {
            public static void main(String[] args) {
                Buffer buffer = ByteBuffer.allocate(7);
                System.out.println("Capacity: " + buffer.capacity());
                System.out.println("Limit: " + buffer.limit());
                System.out.println("Position: " + buffer.position());
                System.out.println("Remaining: " + buffer.remaining());
                System.out.println("Changing buffer limit to 5");
                buffer.limit(5);
                System.out.println("Limit: " + buffer.limit());
                System.out.println("Position: " + buffer.position());
                System.out.println("Remaining: " + buffer.remaining());
                System.out.println("Changing buffer position to 3");
                buffer.position(3);
                System.out.println("Position: " + buffer.position());
                System.out.println("Remaining: " + buffer.remaining());
                System.out.println(buffer);
            }
        }
        ```

    * Buffer详解

        Buffer的众多子类都有相同功能，用ByteBuffer作为例子展开详解Buffer。

        * Buffer创建

            ByteBuffer创建可以用以下静态方法，其内部实现类是HeapByteBuffer

            方法|说明
            --|--
            ByteBuffer allocate(int capacity)|按指定容量大小申请一块新的字节buffer，它有一个后备数组
            ByteBuffer allocateDirect(int capacity)|按指定容量大小申请一块新的直接字节buffer，它有一个后备数组
            ByteBuffer wrap(byte[] array)|将字节数组包装到buffer中，新buffer由数组支持
            ByteBuffer wrap(byte[] array, int offset, int length)|将字节数组包装到buffer中，新buffer由数组支持，position设置为offset，limit设置为offset+length，capacity设置为array.length

            *ByteBuffer通过allocate()和wrap()方法创建的buffer是非直接字节buffer，是支持数组的。*

            假如bufferA创建后，执行方法bufferA.duplicate()生成bufferB，那么bufferA和bufferB各自用一套(mark,position,limit)，但两者的正真数组是一样的，故具有一样的特性:是否只读，是否直接字节buffer。

        * Buffer读写

            ByteBuffer通过put()和get()方法设置和获取数据

            方法|说明
            --|--
            ByteBuffer put(byte x)|position+1，再设置position位置的数据
            ByteBuffer put(int index, byte x)|设置index位置的数据
            ByteBuffer get()|position+1，再获取position位置的数据
            ByteBuffer get()|获取index位置的数据

            *为了效率最大化，可以用以下一些方法设置和获取数据*

            * ByteBuffer put(byte[] src)
            * ByteBuffer put(byte[] src, int offset, int length)
            * ByteBuffer get(byte[] dst)
            * ByteBuffer get(byte[] dst, int offset, int length) 

        * Buffer翻转

            当设置完数据后，如果要把有效的数据输出去，因此可以调用flip()。
            
            * flip():limit置为position，position置为0；

        * Buffer标记

            有时需要再数组中做些标记，然后从特定的标记位置开始做一些操作，因此可以用到mark()和reset()。
            
            * mark():设置mark值为当前position值
            * reset():设置position的值为mark值

        * Buffer子类操作

            * compact():把当前[position,limit]的数据移动到[0,limit-position]，并设置position为limit-position，limit为capacity。

                ```java
                buf.clear(); // Prepare buffer for use
                while (in.read(buf) != -1){//把数据读入到buffer时，都是从position位置开始的
                    buf.flip(); // 相当于buffer进入读模式，limit置为position，position置为0
                    out.write(buf); // 整个待写数据是[0,limit]，从buffer写数据，可能写了部分[0,position]，剩余[position,limit]数据还没写
                    buf.compact(); // 剩余[position,limit]数据整体的移动到最开始位置，并设置position为limit-position，已备下一次读入
                }
                ```

                compact()方法把未写buffer数据移动到buffer的开头，这样一来下一次read()方法调用时追加读数据到buffer数据中，如果没调用compact可能出现覆盖的情况。**这里的read和write方法会影响buff的position值**。
            
            * equals():判断当前buffer是否跟另外一个buffer相等，标准是元素类型相同 [position,limit]长度一致，比[position,limit]这段数据且确保每个元素一样。
            * compareTo():当前buffer与另一个buffer比较，比较两个buffer中[position,limit]这段数据（以短的那一段为准）

        * 字节排序

            假如要向内存地址为a的地方写入数据0x0A0B0C0D，那么这4个字节分别落在哪个地址的内存上呢？这就涉及到字节序的问题了。

            每个数据都有所谓的“有效位（significant byte）”，它的意思是“表示这个数据所用的字节”。例如一个32位整数，它的有效位就是4个字节。而对于0x0A0B0C0D来说，它的有效位从高到低便是0A、0B、0C及0D——这里您可以把它作为一个256进制的数来看（相对于我们平时所用的10进制数）。

            而所谓大字节序（big endian），便是指其“最高有效位（most significant byte）”落在低地址上的存储方式。例如像地址a写入0x0A0B0C0D之后，在内存中的数据便是：

            ![](java/java-io-nio-buffer-big-endian.png)

            而对于小字节序（little endian）来说就正好相反了，它把“最低有效位（least significant byte）”放在低地址上。例如：

            ![](java/java-io-nio-buffer-little-endian.png)

            对于我们常用的CPU架构，如Intel，AMD的CPU使用的都是小字节序，而例如Mac OS以前所使用的Power PC使用的便是大字节序（不过现在Mac OS也使用Intel的CPU了）。此外，除了大字节序和小字节序之外，还有一种很少见的中字节序（middle endian），它会以2143的方式来保存数据（相对于大字节序的1234及小字节序的4321）。

            在java.nio中，字节顺序由ByteOrder类封装。

            ```java
            package java.nio; 
            public final class ByteOrder {
                private String name;
                private ByteOrder(String name) {
                    this.name = name;
                }
                public static final ByteOrder BIG_ENDIAN = new ByteOrder("BIG_ENDIAN");

                public static final ByteOrder LITTLE_ENDIAN  = new ByteOrder("LITTLE_ENDIAN");
                public static ByteOrder nativeOrder() {
                    return Bits.byteOrder();
                }
                public String toString() {
                    return name;
                }
            }
            ```

            ByteOrder类定义了决定从缓冲区中存储或检索多字节数值时使用哪一字节顺序的常量。这个类的作用就像一个类型安全的枚举。它定义了以其本身实例预初始化的两个public区域。只有这两个ByteOrder实例总是存在于JVM中，因此它们可以通过使用--操作符进行比较。如果您需要知道JVM运行的硬件平台的固有字节顺序，请调用静态类函数nativeOrder()。它将返回两个已确定常量中的一个。调用toString()将返回一个包含两个文字字符串BIG_ENDIAN或者LITTLE_ENDIAN之一的String。假设一个叫buffer的ByteBuffer对象处于下图的状态：

            ![](java/java-io-nio-buffer-order.png)

            这段代码： 

            ```java
            //返回一个由缓冲区中位置1-4的byte数据值组成的int型变量的值。实际的返回值取决于缓冲区的当前的比特排序(byte-order)设置
            int value = buffer.getInt(); 
            int value = buffer.order(ByteOrder.BIG_ENDIAN).getInt(); //更具体的写法，这将会返回值0x3BC5315E
            int value = buffer.order(ByteOrder.LITTLE_ENDIAN).getInt(); //返回值0x5E31C53B
            ```

        * 直接字节Buffer

            直接字节Buffer对于JVM处理I/O是最高效的，因此用于channel是高效的。下文做详解

2. Channels

    通道是一个对象，能够打开一个连接到一个硬件设备，一个文件，一个网络socket，一个应用组件，或者是一个可以提供读写或其他I/O操作特性的对象。通道在字节缓冲区和基于操作系统I/O服务的来源或目的地之间高效的传输数据。

    * 通道及其子类

        Java通过java.nio.channels和java.nio.channels.spi来提供通道的，所有的通道实现类都实现java.nio.channels.Channel接口，Channel接口声明了下面的方法

        1. 方法

            方法|说明
            --|--
            void close()|关闭通道
            boolean isOpen()|返回此通道的打开状态

        2. Channel子类(接口类)

            * java.nio.channels.WriteableByteChannel
            * java.nio.channels.ReadableByteChannel
            * java.nio.channels.InterruptibleChannel
            * java.nio.channels.NetworkChannel
            * java.nio.channels.AsynchronousChannel

            1. WriteableByteChannel和ReadableByteChannel

                * WriteableByteChannel接口声明了一个抽象方法 int write(ByteBuffer buffer)，该方法从buffer写入一序列字节到当前通道。
                * ReadableByteChannel接口声明了一个抽象方法 int read(ByteBuffer buffer)，该方法从当前通道中读取一序列字节到buffer中。

            2. InterruptibleChannel

                说明一个通道可以异步的被关闭和被中断，这个接口重载了Channel的close()方法头，添加额外说明：在当前通道发生I/O操作阻塞的任何线程都会受到AsynchronousCloseException异常。主要强调两点：

                * 实现该接口的通道是可异步被关闭的
                * 实现该接口的通道是可异步被中断的

        3. 获取Channel方式

            java.nio.channels包提供了两个方法从流中获取通道

            方法|说明
            --|--
            WritableByteChannel newChannel(OutputStream outputStream)|根据指定的输出流返回一个可写字节通道
            ReadableByteChannel newChannel(InputStream inputStream)|根据指定的输入流返回一个可读字节通道

            几个I/O类被改造支持通道创建，比如
            * java.io.RandomAccessFile声明一个FileChannel getChannel()
            * java.net.Socket声明一个SocketChannel getChannel()

            ```java
            import java.io.IOException;
            import java.nio.ByteBuffer;
            import java.nio.channels.Channels;
            import java.nio.channels.ReadableByteChannel;
            import java.nio.channels.WritableByteChannel;

            public class ChannelDemo {
                public static void main(String[] args) {
                    ReadableByteChannel src = Channels.newChannel(System.in);
                    WritableByteChannel dest = Channels.newChannel(System.out);
                    try {
                        copy(src, dest); // or copyAlt(src, dest);
                    } catch (IOException ioe) {
                        System.err.println("I/O error: " + ioe.getMessage());
                    } finally {
                        try {
                            src.close();
                            dest.close();
                        } catch (IOException ioe) {
                            ioe.printStackTrace();
                        }
                    }
                }

                static void copy(ReadableByteChannel src, WritableByteChannel dest) throws IOException {
                    ByteBuffer buffer = ByteBuffer.allocateDirect(2048);
                    while (src.read(buffer) != -1) {
                        buffer.flip();
                        dest.write(buffer);
                        buffer.compact();
                    }
                    buffer.flip();
                    while (buffer.hasRemaining())
                        dest.write(buffer);
                }

                static void copyAlt(ReadableByteChannel src, WritableByteChannel dest) throws IOException {
                    ByteBuffer buffer = ByteBuffer.allocateDirect(2048);
                    while (src.read(buffer) != -1) {
                        buffer.flip();
                        while (buffer.hasRemaining())
                            dest.write(buffer);
                        buffer.clear();
                    }
                }
            }
            ```

    * Channel详情

        1. 散射/收集 IO

            散射/收集 IO分别定义了两个接口
            * **ScatteringByteChannel** : 继承接口ReadableByteChannel，并定义两个方法
                * long read(ByteBuffer[] dsts)
                * long read(ByteBuffer[] dsts, int offset, int length)
            * **GatheringByteChannel** : 继承接口WritableByteChannel，并定义两个方法
                * long write(ByteBuffer[] dsts)
                * long write(ByteBuffer[] dsts, int offset, int length)

            ```java
            import java.io.FileInputStream;
            import java.io.FileOutputStream;
            import java.io.IOException;
            import java.nio.ByteBuffer;
            import java.nio.channels.Channels;
            import java.nio.channels.GatheringByteChannel;
            import java.nio.channels.ScatteringByteChannel;

            public class GatterAndScatterChannelDemo {
                public static void main(String[] args) throws IOException {
                    ScatteringByteChannel src;
                    FileInputStream fis = new FileInputStream("x.dat");
                    src = (ScatteringByteChannel) Channels.newChannel(fis);
                    ByteBuffer buffer1 = ByteBuffer.allocateDirect(5);
                    ByteBuffer buffer2 = ByteBuffer.allocateDirect(3);
                    ByteBuffer[] buffers = {buffer1, buffer2};
                    src.read(buffers);
                    buffer1.flip();
                    while (buffer1.hasRemaining())
                        System.out.println(buffer1.get());
                    System.out.println();
                    buffer2.flip();
                    while (buffer2.hasRemaining())
                        System.out.println(buffer2.get());
                    buffer1.rewind();
                    buffer2.rewind();
                    GatheringByteChannel dest;
                    FileOutputStream fos = new FileOutputStream("y.dat");
                    dest = (GatheringByteChannel) Channels.newChannel(fos);
                    buffers[0] = buffer2;
                    buffers[1] = buffer1;
                    dest.write(buffers);
                }
            }
            ```
        
        2. 文件通道

            文件通道是一种能够读，写，映射，操作文件的通道。可以通过以下方式获取文件通道
            
            * java.io.FileInputStream.getChannel()
            * java.io.FileOutputStream.getChannel()
            * java.io.RandomAccessFile.getChannel()

            这个抽象类java.nio.channels.FileChannel描述了一个文件通道。该类继承AbstractInterruptibleChannel抽象类，因此可中断。另外，该类实现了接口SeekableByteChannel，GatheringByteChannel，ScatteringByteChannel，因此可以在基础文件上写入，读取，散射或收集I/O。

            方法|说明
            --|--
            void force(boolean metadata)|所有对文件通道的更新都提交到对应的文件。metadata为true则更新文件metadata，否则不更新
            long position()|返回这个文件通道的文件位置
            FileChannel position(long newPosition)|设置这个文件通道的文件位置到newPosition
            int read(ByteBuffer buffer)|从文件通道中读取字节到指定的buffer
            int read(ByteBuffer dst, long position)|从文件通道中读取字节到指定的buffer，并从position位置开始
            long size()|返回基础文件的大小(按字节算)
            FileChannel truncate(long size)|将此通道的文件截断为给定的大小。size大于当前文件大小，丢弃文件新末尾之外的任何字节，否则文件不做改变，且position设置size
            int write(ByteBuffer buffer)|从指定的buffer写入到文件通道
            int write(ByteBuffer src, long position)|从指定的buffer写入到文件通道，并从position位置开始

            ```java
            import java.io.IOException;
            import java.io.RandomAccessFile;
            import java.nio.ByteBuffer;
            import java.nio.channels.FileChannel;

            public class FileChannelDemo {
                public static void main(String[] args) throws IOException {
                    RandomAccessFile raf = new RandomAccessFile("temp", "rw");
                    FileChannel fc = raf.getChannel();
                    long pos;
                    System.out.println("Position = " + (pos = fc.position()));
                    System.out.println("size: " + fc.size());
                    String msg = "This is a test message.";
                    ByteBuffer buffer = ByteBuffer.allocateDirect(msg.length() * 2);
                    buffer.asCharBuffer().put(msg);
                    fc.write(buffer);
                    fc.force(true);
                    System.out.println("position: " + fc.position());
                    System.out.println("size: " + fc.size());
                    buffer.clear();
                    fc.position(pos);
                    fc.read(buffer);
                    buffer.flip();
                    while (buffer.hasRemaining())
                        System.out.print(buffer.getChar());
                }
            }
            ```

            **文件锁定**

            在Java1.4后支持了锁定一个文件的所有或部分，文件锁分为独占锁和共享锁，读写锁的三种状态：

            1. 当读写锁是写加锁状态时，在这个锁被解锁之前，所有试图对这个锁加锁的线程都会被阻塞

            2. 当读写锁在读加锁状态时，所有试图以读模式对它进行加锁的线程都可以得到访问权，但是以写模式对它进行加锁的线程将会被阻塞
            
            3. 当读写锁在读模式的锁状态时，如果有另外的线程试图以写模式加锁，读写锁通常会阻塞随后的读模式锁的请求，这样可以避免读模式锁长期占用，而等待的写模式锁请求则长期阻塞。

            **文件锁定注意要点**

            * 如果操作系统不支持共享锁，那么对共享锁的请求将转为独占锁
            * 锁是基于每个文件应用的。它们不是基于每个线程或每个通道应用的。运行在同一JVM上的两个线程通过不同的通道请求对同一文件区域的独占锁，并被授予访问权限。但是，如果这些线程运行在不同的jvm上，第二个线程就会阻塞。锁最终由操作系统的文件系统仲裁，而且几乎总是在进程级。它们不在线程级别仲裁。锁与文件关联，而不是与文件句柄或通道关联

            获取独占锁和共享锁的方法

            方法|说明
            --|--
            FileLock lock()|获取此文件通道的基础文件上的独占锁
            FileLock lock(long position, long size, boolean shared)|从指定位置开始的一块区域，获取独占锁(shared为false)或共享锁(shared为true)
            FileLock tryLock()|以非阻塞的方式尝试获取此文件通道的基础文件上的独占锁
            FileLock tryLock(long position, long size, boolean shared)|从指定位置开始的一块区域，以非阻塞的方式尝试获取独占锁(shared为false)或共享锁(shared为true)

            上面的四个方法都返回了一个FileLock的实例，该实例的方法如下
            
            方法|说明
            --|--
            FileChannel channel()|返回文件锁的文件通道
            void close()|调用了release()方法来释放锁
            boolean isShared()|判断文件锁是不是共享锁
            boolean isValid()|只有文件锁被释放或关联的文件通道被关闭则返回true，否则返回false
            void release()|释放锁

            ```java
            FileLock lock = fileChannel.lock();
            try{
            // interact with the file channel
            }catch (IOException ioe){
            // handle the exception
            }finally{
                lock.release();
            }
            ```

            







3. Selectors

   

4. Regular Expression

5. Charsets

6. Formatter

    

原文:book/Java I-O, NIO and NIO.2.pdf
https://blog.csdn.net/will_awoke/article/details/25803725