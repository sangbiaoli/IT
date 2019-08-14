## ByteBuf

1. ByteBuf API

    netty数据处理API两大组件：抽象类ByteBuf和接口ByteBufHolder。

    ByteBuf API的优势：
    
    * 它可扩展到用户定义的缓冲区类型。
    * 透明的零拷贝是通过内置的复合缓冲区类型实现的。
    * 容量根据需要进行扩展(与JDK StringBuilder一样)。
    * 在读写模式之间切换不需要调用ByteBuffer的flip()方法。
    * 读和写采用不同的指标。
    * 支持方法链接。
    * 支持引用计数。
    * 池的支持。

2. 类ByteBuf-Netty数据容器

    1. 工作原理

        ByteBuf维护两个不同的索引:一个用于读取，另一个用于写入。当您从ByteBuf读取数据时，它的readerIndex会随着读取的字节数而增加。
        类似地，当您写入ByteBuf时，它的writerIndex会递增。ByteBuf的布局和状态

        ![](netty/netty-bytebuf-index.png)

    2. ByteBuf使用模式

        * 堆缓冲区

            最常用的ByteBuf模式将数据存储在JVM。这种模式称为支持数组，在不使用池的情况下提供快速分配和释放位置。下面显示的这种方法非常适合处理遗留数据的情况。

            ```java
            ByteBuf heapBuf = ...;
            if (heapBuf.hasArray()) {      //检查是否支持数组     
                byte[] array = heapBuf.array();     //获取数组引用                    
                int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();
                int length = heapBuf.readableBytes();    
                handleArray(array, offset, length);
            }
            ```

        * 直接缓冲区

            JDK 1.4中使用NIO引入的ByteBuffer类允许JVM实现通过本机调用分配内存。这样做的目的是避免在每次调用本机I/O操作之前(或之后)将缓冲区的内容复制到(或从)中间缓冲区。

            直接缓冲区的主要缺点是，与基于堆的缓冲区相比，它们在分配和释放方面花费更多。如果使用遗留代码，可能还会遇到另一个缺点:由于数据不在堆上，可能需要复制，如下所示。

            ```java
            ByteBuf directBuf = ...;
            if (!directBuf.hasArray()) {              
                int length = directBuf.readableBytes();             
                byte[] array = new byte[length];                    
                directBuf.getBytes(directBuf.readerIndex(), array); 
                handleArray(array, 0, length);        
            }
            ```

            显然，这比使用支持数组要多做一点工作，所以如果您事先知道容器中的数据将作为数组访问，您可能更愿意使用堆内存。

        * 复合缓冲

            第三种模式(也是最后一种模式)使用复合缓冲区，它显示了多个ByteBuf的聚合视图。在这里，您可以根据需要添加和删除ByteBuf实例，这在JDK的ByteBuffer实现中是完全没有的。

            Netty使用ByteBuf的子类CompositeByteBuf实现了这种模式，它提供了一个多个缓冲区作为单个合并缓冲区的虚拟表示。

            ![](netty/netty-bytebuf-composite.png)

            * 使用ByteBuffer的复合缓冲区模式

                ```java
                // Use an array to hold the message parts
                ByteBuffer[] message = new ByteBuffer[] { header, body };
                // Create a new ByteBuffer and use  copy to merge the header and body
                ByteBuffer message2 = ByteBuffer.allocate(header.remaining() + body.remaining());
                message2.put(header);
                message2.put(body);
                message2.flip();
                ```

            * 使用CompositeByteBuf的复合缓冲模式

                ```java
                CompositeByteBuf messageBuf = Unpooled.compositeBuffer();
                ByteBuf headerBuf = ...; // can be backing or direct
                ByteBuf bodyBuf = ...;   // can be backing or direct
                messageBuf.addComponents(headerBuf, bodyBuf);              
                .....
                messageBuf.removeComponent(0); // remove the header     
                for (ByteBuf buf : messageBuf) {                   
                    System.out.println(buf.toString());
                }
                ```

            * 访问CompositeByteBuf中的数据

                ```java
                CompositeByteBuf compBuf = Unpooled.compositeBuffer();
                int length = compBuf.readableBytes();                 
                byte[] array = new byte[length];                       
                compBuf.getBytes(compBuf.readerIndex(), array);       
                handleArray(array, 0, array.length);
                ```
        
3. Byte级别操作

    1. 随机存储索引
    2. 序列存储索引
    3. 可废弃字节
    4. 可读字节
    5. 可写字节
    6. 索引管理
    7. 搜索操作
    8. 衍生缓冲区
    9. 读/写操作
    10. 更多操作

4. 接口ByteBufHolder

5. ByteBuf分配

    1. 请求式：接口ByteBufAllocator
    2. 非池化缓冲区
    3. 类ByteBufUtil
