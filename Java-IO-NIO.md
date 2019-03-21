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

    

3. Selectors

   

4. Regular Expression

5. Charsets

6. Formatter

    

原文:book/Java I-O, NIO and NIO.2.pdf
https://blog.csdn.net/will_awoke/article/details/25803725