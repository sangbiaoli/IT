1. 接口与抽象类的区别？ 
    1. 抽象类

        抽象类是用来捕捉子类的通用特性的 。它不能被实例化，只能被用作子类的超类。抽象类是被用来创建继承层级里子类的模板。以JDK中的GenericServlet为例：
        ```java
        public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
            // abstract method
            abstract void service(ServletRequest req, ServletResponse res);

            void init() {
                // Its implementation
            }
            // other method related to Servlet
        }
        ```
        当HttpServlet类继承GenericServlet时，它提供了service方法的实现：
        ```java
        public class HttpServlet extends GenericServlet {
            void service(ServletRequest req, ServletResponse res) {
                // implementation
            }

            protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
                // Implementation
            }

            protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
                // Implementation
            }

            // some other methods related to HttpServlet
        }
        ```
    2. 接口

        接口是抽象方法的集合。如果一个类实现了某个接口，那么它就继承了这个接口的抽象方法。这就像契约模式，如果实现了这个接口，那么就必须确保使用这些方法。接口只是一种形式，接口自身不能做任何事情。以Externalizable接口为例：
        ```java
        public interface Externalizable extends Serializable {

            void writeExternal(ObjectOutput out) throws IOException;

            void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
        }
        ```
        当你实现这个接口时，你就需要实现上面的两个方法：
        ```java
        public class Employee implements Externalizable {

            int employeeId;
            String employeeName;

            @Override
            public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
                employeeId = in.readInt();
                employeeName = (String) in.readObject();

            }

            @Override
            public void writeExternal(ObjectOutput out) throws IOException {

                out.writeInt(employeeId);
                out.writeObject(employeeName);
            }
        }
        ```
    3. 抽象类和接口的对比

        参数|抽象类|接口
        --|--|--
        默认的方法实现|它可以有默认的方法实现|接口完全是抽象的。它根本不存在方法的实现
        实现|子类使用extends关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。|子类使用关键字implements来实现接口。它需要提供接口中所有声明的方法的实现
        构造器|抽象类可以有构造器|接口不能有构造器
        与正常Java类的区别|除了你不能实例化抽象类之外，它和普通Java类没有任何区别|接口是完全不同的类型
        访问修饰符|抽象方法可以有public、protected和default这些修饰符|接口方法默认修饰符是public。你不可以使用其它修饰符。
        main方法|抽象方法可以有main方法并且我们可以运行它|接口没有main方法，因此我们不能运行它。
        多继承|抽象方法可以继承一个类和实现多个接口|接口只可以继承一个或多个其它接口
        速度|它比接口速度要快|接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。
        添加新方法|如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。|如果你往接口中添加方法，那么你必须改变实现该接口的类。
    4. 什么时候使用抽象类和接口

        1. 如果你拥有一些方法并且想让它们中的一些有默认实现，那么使用抽象类吧。
        1. 如果你想实现多重继承，那么你必须使用接口。由于Java不支持多继承，子类不能够继承多个类，但可以实现多个接口。因此你就可以使用接口来解决它。
        1. 如果基本功能在不断改变，那么就需要使用抽象类。如果不断改变基本功能并且使用接口，那么就需要改变所有实现了该接口的类。

    原文：http://www.importnew.com/12399.html

2. Java中的异常有哪几类？分别怎么使用？ 

    1. 异常类有分为编译时异常和运行时异常
        1. 编译时异常:写代码的时候就会提醒你有异常

            1. IOException
            1. SQLException
            1. CloneNotSupportedException
            1. parseException

        2. 运行时异常:java.lang.RuntimeException,运行的时候会在控制台提示异常

            1. NullPointerException: 空指针异常,一般出现于数组,空对象的变量和方法
            1. ArrayIndexOutOfBoundsException: 数组越界异常
            1. ArrayStoreException: 数据存储异常
            1. NoClassDefFoundException: java运行时系统找不到所引用的类
            1. ArithmeticException: 算数异常,一般在被除数是0中
            1. ClassCastException: 类型转换异常
            1. IllegalArgumentException: 非法参数异常
            1. IllegalThreadStateException: 非法线程状态异常
            1. NumberFormatException: 数据格式异常
            1. OutOfMemoryException: 内存溢出异常
            1. PatternSyntaxException: 正则异常

        3. 自定义异常:

            自定义一个类,继承某个异常类
            1. 如果继承的是Exception那么就定义了一个编译时异常
            1. 如果继承的是RuntimeException或者其子类,那么就定义了一个运行时异常

    1. 怎么使用
        1. 一种是在方法中声名异常,谁调用就把异常抛给谁

        2. 一种是使用try{}..catch{}块处理异常
            1. 如果多个异常处理的方式不同,可以用多个catch处理
            1. 如果所有异常处理方式一样,可以捕获一个父类异常进行统一的处理
            1. 如果多个异常分成了不同的组,那么同一组异常之间可以使用|隔开(jdk1.7开始)
            1. jdk1.7还增加了增强tr(){}catch(){},通常用于自动关流

    1. 异常知识扩展
        1. Throwable类是所有异常的超类,有两个子类,分为Error和Exception
            1. Error:错误是无法处理的,只能更改代码,就像一个人得癌症一样
            1. Exception:异常是可以处理的,就像是感冒一样,吃药就能好

        2. 在方法重写的时候
            1. 子类抛出的编译时异常不能超过父类编译时异常范围
            1. 子类不能抛出比父类更多的编译时异常(这里是指抛出异常的范围不能更大,但个数可以更多)
            编译时异常随你抛

    原文：https://blog.csdn.net/JetaimeHQ/article/details/83031899

3. 常用的集合类有哪些？比如List如何排序？ 

    ![](ali/ali-java-collection.png)

    #### Java中常用的集合类

    * **Collection**
        * **List**
            * ArrayList
            * Vector
            * LinkedList
        * **Set**
            * HashSet
            * LinkedHashSet
            * **SortedSet**
                * TreeSet
    * **Map**:
        * HashMap
        * LinkedHashMap
        * **SortedMap**
            * TreeMap

    1. List
        1. ArrayList

            ArrayList  是一个动态数组，以一个数组的格式进行存储，但是可以动态增长，继承自AbstractList，实现了List接口

            ArrayList是一个非线程安全的类，使用时最好在一个线程中操作，构造函数可以提供一个数组的初始化大小，不提供默认按16来进行数组的初始化，往ArrayList增加元素时，都会先判断是否还有空间存在元素，如果没有空间，则会重新申请一个

            (((以前长度*3)/2)+1)大小的数组空间，并将原来的元素全部copy到新的数组中，这样会带来效率上的损耗。建议在初始化时，能够确定存储空间的尽量提供合适的初始化数组大小，避免因为数组动态扩容带来的效率损耗。
        2. Vector

            Vector 基本上很少用到了，和ArrayList很相似，唯一的不同是Vector的方法都是线程安全的，而Array List是非线程安全的，从这点上来说，ArrayList的效率要比Vector的效率高，另外Vector是以2倍的方式扩展数组容量的

        3. LinkedList

            LinkedList 同时实现了List，Deque,所以也可以用来作为双向队列使用。LinkedList是以双向列表存储的，它是按照元素的先后顺序进行存储的，所以访问也是按照顺序来访问的。

        JDK7以前用collections.sort(list,Comparator).

        JDK8直接用List.sort(Comparator).

    2. Set  
        Set 继承了Collection，Set不保存重复的元素，存入Set里的元素都是唯一的，区分存入的元素是否重复是通过调用Equals方法来进行判断，所有存入Set中的元素类必须实现Equals方法

        1. HashSet

            HashSet是讲存入Set中的元素以Hash链表的方式存储起来，所有元素存入Set中的位置通过调用hashCode方法获取hash值来决定，所以需要设计好的hashcode方法尽量将存入的值散列开有利于提升HashSet的访问效率。

        2. LinkedHashSet

            继承与HashSet,和HashSet的唯一区别是维护了一个双向列表来维护元素的顺序，所有访问是按顺序访问

        3. TreeSet
            TreeSet继承于AbstractSet，并且实现了NavigableSet接口
            TreeSet实现了一个顺序访问的不重复元素的Set，底层使用红黑树进行数据的存储，来加快访问的速度

        4. EnumSet
            EnumSet只能用来存放Enum类型的数据，也不允许重复数据，性能是最好的
    3. Map

        Map和Set的区别是Set只有值，而Map是一个键值对<key,value>，Set不存重复的元素，Map中的key不能相同

        1. HashMap

            和HashSet相似，用hash链表来存储，讲键值对作为一个元素存储

        2. LinkedHashMap

            类似于HashMap，但是迭代遍历它时，取得“键值对”的顺序是其插入次序，或者是最近最少使用(LRU)的次序。只比HashMap慢一点。而在迭代访问时发而更快，因为它使用链表维护内部次序

        3. TreeMap

            基于红黑树数据结构的实现。查看“键”或“键值对”时，它们会被排序(次序由Comparabel或Comparator决定)。TreeMap的特点在于，你得到的结果是经过排序的。TreeMap是唯一的带有subMap()方法的Map，它可以返回一个子树。

        4. WeakHashMap

            弱键(weak key)Map，Map中使用的对象也被允许释放: 这是为解决特殊问题设计的。如果没有map之外的引用指向某个“键”，则此“键”可以被垃圾收集器回收。

        原文：https://blog.csdn.net/peterzhihua/article/details/71260811

4. ArrayList和LinkedList内部的实现大致是怎样的？他们之间的区别和优缺点？ 

    ArrayList和LinkedList都是实现了List接口的容器类，用于存储一系列的对象引用。他们都可以对元素的增删改查进行操作。

    对于ArrayList，它在集合的末尾删除或添加元素所用的时间是一致的，但是在列表中间的部分添加或删除时所用时间就会大大增加。但是它在根据索引查找元素的时候速度很快。

    对于LinkedList则相反，它在插入、删除集合中任何位置的元素所花费的时间都是一样的，但是它根据索引查询一个元素的时候却比较慢。

    ArrayList和LinkedList的大致区别：

    1. ArrayList是实现了基于动态数组的数据结构，LinkedList是基于链表结构。
    2. 对于随机访问的get和set方法，ArrayList要优于LinkedList，因为LinkedList要移动指针。
    3. 对于新增和删除操作add和remove，LinkedList比较占优势，因为ArrayList要移动数据。

    它们在性能上的有缺点：

    1. 对ArrayList和LinkedList而言，在列表末尾增加一个元素所花的开销都是固定的。对 ArrayList而言，主要是在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配；而对LinkedList而言，这个开销是 统一的，分配一个内部Entry对象。
    2. 在ArrayList集合中添加或者删除一个元素时，当前的列表所所有的元素都会被移动。而LinkedList集合中添加或者删除一个元素的开销是固定的。
    3. LinkedList集合不支持 高效的随机随机访问（RandomAccess），因为可能产生二次项的行为。
    4. ArrayList的空间浪费主要体现在在list列表的结尾预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗相当的空间

    所以在我们进行对元素的增删查操作的时候，进行 查操作时用ArrayList，进行增删操作的时候最好用LinkedList。

    原文：https://blog.csdn.net/qq_32679815/article/details/78907437 
5. 内存溢出是怎么回事？请举一个例子？ 
    #### 内存泄露

    什么是内存泄露，因为执行程序指令，引入数据，装载数据等等，都是需要内存空间的，内存空间也是有限的。

    正常的程序执行，肯定是开辟内存空间，运行程序指令(输入、输出)，程序结束，释放空间。这样可以保证内存的往复利用。

    但是，有时候程序设计有漏洞或是不够严谨的话，导致占用的内存没有得到释放，那么这一块内存就没办法再利用了，这就是内存泄露。

    当不断出现内存泄露的时候，被占用的内存空间越来越多，最终到下一次需要使用内存空间的时候就会这样——空间不够了，溢出了。

    #### Java的内存溢出

    对于Java，内存溢出分三种情况。
    1. OutOfMemoryError： PermGen space

        Permanent Generation space 这个区域主要用来保存加来的Class的一些信息，在程序运行期间属于永久占用的，Java的GC不会对他进行释放，所以如果启动的程序加载的信息比较大，超出了这个空间的大小，就会发生溢出错误；

        解决的办法无非就是增加空间分配了——增加java虚拟机中的XX:PermSize和XX:MaxPermSize参数的大小，其中
        * XX:PermSize是初始永久保存区域大小
        * XX:MaxPermSize是最大永久保存区域大小。

    2. OutOfMemoryError：Java heap space

        heap 是Java内存中的堆区，主要用来存放对象，当对象太多超出了空间大小，GC又来不及释放的时候，就会发生溢出错误。
        Java中对象的创建是可控的，但是对象的回收是由GC自动的，一般来说，当已存在对象没有引用(即不可达)的时候，GC就会定时的来回收对象，释放空间。但是因为程序的设计问题，导致对象可达但是又没有用(即前文提到的内存泄露)，当这种情况越来越多的时候，问题就来了。
        针对这个问题，我们需要做一下两点：
        1. 检查程序，减少大量重复创建对象的死循环，减少内存泄露。
        2. 增加Java虚拟机中Xms（初始堆大小）和Xmx（最大堆大小）参数的大小。

    3. StackOverFlowError

        stack是Java内存中的栈空间，主要用来存放方法中的变量，参数等临时性的数据的，发生溢出一般是因为分配空间太小，或是执行的方法递归层数太多创建了占用了太多栈帧导致溢出。
        针对这个问题，除了修改配置参数-Xss参数增加线程栈大小之外，优化程序是尤其重要。

    原文：https://www.jianshu.com/p/8c31ef19983a

6. ==和equals的区别？ 
    1. 对于==，比较的是值是否相等
        * 如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等；
        * 如果作用于引用类型的变量，则比较的是所指向的对象的地址

    2.  对于equals方法
        * equals方法不能作用于基本数据类型的变量，equals继承Object类，比较的是是否是同一个对象

        * 如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址；

        * 诸如String、Date等类对equals方法进行了重写的话，比较的是所指向的对象的内容。

    ```java
    public class TestString {
        public static void main(String[] args) {
        String s1 = "Monday";
        String s2 = "Monday";   //s1 == s2，因为指向String缓冲池的同一个地址
        String s3 = new String("Monday");//s2 != s3,但是s2 equals s3，s3不存放在String缓冲池，而是一个新地址
        s3 = s3.intern();   //s2 == s3同时s2 equals s3，intern()方法检查字符串池里是否存在"Monday"这么一个字符串，如果存在，就返回池里的字符串；如果不存在，该方法会 把"Monday"添加到字符串池中，然后再返回它的引用
    }
    ```

    原文：https://blog.csdn.net/qq_27471405/article/details/81010094

7. hashCode方法的作用？ 
    #### 前言
    Object提供给我们了一个Native的方法“public native int hashCode();”，本文讲讲Hash是什么以及HashCode的作用

    #### Hash
    先用一张图看下什么是Hash

    ![](ali/ali-java-hashCode.jpg)

    Hash是散列的意思，就是把任意长度的输入，通过散列算法变换成固定长度的输出，该输出就是散列值。关于散列值，有以下几个关键结论：
    1. 如果散列表中存在和散列原始输入K相等的记录，那么K必定在f(K)的存储位置上

    2. 不同关键字经过散列算法变换后可能得到同一个散列地址，这种现象称为碰撞

    3. 如果两个Hash值不同（前提是同一Hash算法），那么这两个Hash值对应的原始输入必定不同

    #### HashCode

    然后讲下什么是HashCode，总结几个关键点：

    1. HashCode的存在主要是为了查找的快捷性，HashCode是用来在散列存储结构中确定对象的存储地址的

    2. 如果两个对象equals相等，那么这两个对象的HashCode一定也相同

    3. 如果对象的equals方法被重写，那么对象的HashCode方法也尽量重写

    4. 如果两个对象的HashCode相同，不代表两个对象就相同，只能说明这两个对象在散列存储结构中，存放于同一个位置

    #### HashCode有什么用

    回到最关键的问题，HashCode有什么用？不妨举个例子：

    1. 假设内存中有0 1 2 3 4 5 6 7 8这9个位置，如果我有个字段叫做ID，那么我要把这个字段存放在以上8个位置之一，如果不用HashCode而任意存放，那么当查找时就需要到8个位置中去挨个查找

    2. 使用HashCode则效率会快很多，把ID的HashCode%8，然后把ID存放在取得余数的那个位置，然后每次查找该类的时候都可以通过ID的HashCode%8求余数直接找到存放的位置了

    3. 如果ID的HashCode%8算出来的位置上本身已经有数据了怎么办？这就取决于算法的实现了，比如ThreadLocal中的做法就是从算出来的位置向后查找第一个为空的位置，放置数据；HashMap的做法就是通过链式结构连起来。反正，只要保证放的时候和取的时候的算法一致就行了。

    4. 如果ID的HashCode%8相等怎么办（这种对应的是第三点说的链式结构的场景）？这时候就需要定义equals了。先通过HashCode%8来判断类在哪一个位置，再通过equals来在这个位置上寻找需要的类。对比两个类的时候也差不多，先通过HashCode比较，假如HashCode相等再判断equals。如果两个类的HashCode都不相同，那么这两个类必定是不同的。

    举个实际的例子Set。我们知道Set里面的元素是不可以重复的，那么如何做到？Set是根据equals()方法来判断两个元素是否相等的。
    
    比方说Set里面已经有1000个元素了，那么第1001个元素进来的时候，最多可能调用1000次equals方法，如果equals方法写得复杂，对比的东西特别多，那么效率会大大降低。使用HashCode就不一样了，
    
    比方说HashSet，底层是基于HashMap实现的，先通过HashCode取一个模，这样一下子就固定到某个位置了，如果这个位置上没有元素，那么就可以肯定HashSet中必定没有和新添加的元素equals的元素，就可以直接存放了，都不需要比较；如果这个位置上有元素了，逐一比较，比较的时候先比较HashCode，HashCode都不同接下去都不用比了，肯定不一样，HashCode相等，再equals比较，没有相同的元素就存，有相同的元素就不存。如果原来的Set里面有相同的元素，只要HashCode的生成方式定义得好（不重复），不管Set里面原来有多少元素，只需要执行一次的equals就可以了。这样一来，实际调用equals方法的次数大大降低，提高了效率。

    原文：https://www.cnblogs.com/xrq730/p/4842028.html

8. NIO是什么？适用于何种场景？ 
    #### Java NIO和IO之间的主要区别
    下表总结了Java NIO和IO之间的主要区别。 我将在表格后面的部分详细介绍每个区别。
    IO|NIO
    --|--
    面向流|利用缓冲区
    阻塞IO|非阻塞IO
    ||Selectors

    #### 流导向vs缓冲导向
    Java NIO和IO之间的第一大区别是IO是面向流的，其中NIO是面向缓冲区的。 那么，这是什么意思？

    面向流的Java IO意味着您一次从流中读取一个或多个字节。 你所做的读字节取决于你。 他们没有被缓存在任何地方。 而且，您不能前后移动数据流。 如果您需要前后移动从流中读取的数据，则需要先将其缓存在缓冲区中。

    Java NIO的面向缓冲区的方法略有不同。 数据被读入一个缓冲区，稍后进行处理。 您可以根据需要前后移动缓冲区。 这给你在处理过程中更多的灵活性。 但是，您还需要检查缓冲区是否包含您需要的所有数据，以便对其进行全面处理。 而且，您需要确保在将更多数据读入缓冲区时，不会覆盖尚未处理的缓冲区中的数据。

    #### 阻塞vs非阻塞IO
    Java IO的各种流都被阻塞。 这意味着，当一个线程调用一个read()或write()时，该线程被阻塞，直到有一些数据要读取，或者数据被完全写入。 线程在此期间不能做其他事情。

    Java NIO的非阻塞模式使得线程可以请求从一个通道读取数据，并且只获取当前可用的数据，或者根本没有任何数据可用。 线程可以继续使用别的东西，而不是在数据可用于读取之前保持阻塞状态。
    无阻塞写入也是如此。 一个线程可以请求将一些数据写入一个通道，但不要等待它被完全写入。 线程然后可以继续，同时做其他事情。

    线程在没有被IO阻塞的情况下花费空闲时间，通常是在其他通道上同时执行IO。 也就是说，一个线程现在可以管理多个输入输出通道。

    #### Selectors
    Java NIO的选择器允许单线程监视多个输入通道。 您可以使用选择器注册多个通道，然后使用单个线程“选择”可用于处理的输入通道，或者选择准备写入的通道。 这个选择器机制使单个线程可以轻松管理多个通道。

    #### NIO和IO如何影响应用程序设计
    无论您选择NIO还是IO作为您的IO工具包，都可能会影响您应用程序设计的以下方面：
    1. API调用NIO或IO类。
    2. 数据的处理。
    3. 用于处理数据的线程数。

    #### API调用
    当然，使用NIO时的API调用看起来不同于使用IO时的调用。 这并不奇怪。 而不只是从例如数据字节读取字节。 一个InputStream，数据必须首先被读入一个缓冲区，然后从那里被处理。
    #### 数据处理
    当使用纯粹的NIO设计时，数据的处理也会受到IO设计的影响。
    在IO设计中，您从InputStream或Reader读取字节的数据字节。 想象一下，你正在处理一系列基于行的文本数据。 例如：
    ```
    Name: Anna
    Age: 25
    Email: anna@mailserver.com
    Phone: 1234567890
    ```
    这条文本行可以像这样处理：
    ```java
    InputStream input = ... ; // get the InputStream from the client socket

    BufferedReader reader = new BufferedReader(new InputStreamReader(input));

    String nameLine   = reader.readLine();
    String ageLine    = reader.readLine();
    String emailLine  = reader.readLine();
    String phoneLine  = reader.readLine();
    ```
    注意处理状态是由程序执行的程度决定的。 换句话说，一旦第一个reader.readLine()方法返回，您肯定知道已经读取了一整行文本。 readLine()阻塞直到读完整行，这就是为什么。 你也知道这行包含名字。 同样，当第二个readLine()调用返回时，您知道该行包含年龄等

    正如你所看到的，只有当有新的数据要读取时，程序才会进行，并且每一步你都知道数据是什么。 一旦正在执行的线程通过读取代码中的某段数据，线程就不会在数据中倒退（大部分不是）。 这个原理在这个图中也有说明：

    ![](ali/ali-java-nio-thread.png)

    NIO的实现看起来不同。 这是一个简单的例子：
    ```java
    ByteBuffer buffer = ByteBuffer.allocate(48);

    int bytesRead = inChannel.read(buffer);
    ```
    注意从通道读取字节到ByteBuffer的第二行。 当该方法调用返回时，您不知道所需的所有数据是否在缓冲区内。 所有你知道的是，缓冲区包含一些字节。 这使得处理有点困难。

    想象一下，如果在第一次读取（缓冲区）调用之后，所有读入缓冲区的内容都是半行。 例如，"Name: An"。 你能处理这些数据吗？ 不是真的。 你需要等待，直到完整的一行数据已经进入缓冲区，才有意义处理任何数据。

    那么如何知道缓冲区中是否包含足够的数据才能处理？ 那么，你没有。 唯一的办法就是查看缓冲区中的数据。 结果是，您可能需要多次检查缓冲区中的数据，才能知道所有数据是否在内部。 这既是低效的，而且在程序设计方面可能变得混乱。 例如：
    ```java
    ByteBuffer buffer = ByteBuffer.allocate(48);

    int bytesRead = inChannel.read(buffer);

    while(! bufferFull(bytesRead) ) {
        bytesRead = inChannel.read(buffer);
    }
    ```
    bufferFull()方法必须跟踪有多少数据被读入缓冲区，并根据缓冲区是否已满而返回true或false。 换句话说，如果缓冲区准备好处理，则认为已满。
    bufferFull()方法扫描缓冲区，但必须保持缓冲区处于与调用bufferFull()方法之前相同的状态。 否则，读入缓冲区的下一个数据可能无法在正确的位置读入。 这不是不可能的，但这是另一个需要注意的问题。
    如果缓冲区已满，可以进行处理。 如果它没有满，那么你可能能够部分处理任何数据，如果这在你的特定情况下是有意义的。 在许多情况下，它不是。
    图中显示了is-data-in-buffer-ready循环：

    ![](ali/ali-java-nio-buffer-ready.png)

    #### 概要
    NIO允许您仅使用一个（或几个）线程来管理多个通道（网络连接或文件），但代价是解析数据可能比从阻塞流读取数据时要复杂一些。

    如果你需要管理数以千计的同时打开的连接，每一个只发送一点点的数据，例如聊天服务器，在NIO中实现服务器可能是一个优势。 同样，如果你需要保持与其他计算机的大量开放连接，例如 在P2P网络中，使用单个线程来管理所有出站连接可能是一个优势。 这一个线程，多个连接设计如下图所示：

    ![](ali/ali-java-nio-thread-conn.png)

    如果您的连接带宽非常高，一次发送大量数据，那么传统的IO服务器实现可能是最合适的。 该图说明了一个经典的IO服务器设计：

    ![](ali/ali-java-nio-thread-conn-thread.png)

    原文：https://www.jianshu.com/p/18f9d350b5c9?mType=Group

9. HashMap实现原理，如何保证HashMap的线程安全？ 
    1. 前言

        能用图说清楚的，就坚决不用代码。能用代码撸清楚的，就坚决不写解释（不是不写注释哦）。
        
        以下所有仅针对JDK 1.7及之前中的HashMap。

    2. 数据结构

        HashMap内部通过维护一个Entry<K, V>数组（变量为table），来实现其基本功能，而Entry<K, V>是HashMap的内部类，其主要作用便是存储键值对，其数据结构大致如下图所示。

        ![](ali/ali-java-hashMap-structure.jpg)

        从Entry的数据结构可以看出，多个Entry是可以形成一个单向链表的，HashMap中维护的Entry<K, V>数组（之后简称为Entry数组，或table，容易区分）其实就是存储的一系列Entry<K, V>链表的表头。那么HashMap中存储数据table数组的数据结构，大致可以如下图所示（假设只有部分数据）。

        ![](ali/ali-java-hashMap-store.jpg)

        注：Entry数组的默认长度为16，负载因子为0.75。

        将上图中的每一行，称为桶（bucket），那么table的索引便是bucketIndex。而HashMap中的插入、获取、删除等操作最主要的便是对table和桶（bucket）的操作。下面将主要通过插入操作，看其数据结构的变化。

    3. 插入

        对于上图中的数据结构，插入操作便是将要插入的键 - 值（key - value）对根据key计算hash值来选择具体的存储位置。

        插入函数的源码如下（以Mark开头的或者中文注释，非JDK源码中的注释，下同）：
        ```java
        public V put(K key, V value) {
            // Mark A Begin
            if (table == EMPTY_TABLE) {
                inflateTable(threshold);
            }
            if (key == null)
                return putForNullKey(value);
            // Mark A End

            int hash = hash(key); // 计算hash值
            int i = indexFor(hash, table.length); // 计算桶的位置索引（bucketIndex）

            // Mark B begin
            for (Entry<K,V> e = table[i]; e != null; e = e.next) {
                Object k;
                if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                    V oldValue = e.value;
                    e.value = value;
                    e.recordAccess(this);
                    return oldValue;
                }
            }
            // Mark B end

            modCount++; // 记录修改次数，迭代的时候会据此判断是否有被修改
            addEntry(hash, key, value, i);
            return null;
        }
        ```
        在上面的代码中，代码段A（Mark A Begin - Mark A End，下同）的主要作用是如果table为空则初始化数组以及插入key为null时的操作，代码段B则是插入相同key时覆盖原有的值，并返回原有的值。这里重点关注的是addEntry(hash, key, value, i)方法。

        addEntry方法源码如下：
        ```java
        void addEntry(int hash, K key, V value, int bucketIndex) {
            if ((size >= threshold) && (null != table[bucketIndex])) {
                // 扩充table数组的大小
                resize(2 * table.length);
                // 重新计算hash值
                hash = (null != key) ? hash(key) : 0;
                // 重新计算桶的位置索引
                bucketIndex = indexFor(hash, table.length);
            }

            createEntry(hash, key, value, bucketIndex);
        }
        ```
        createEntry方法源码如下：
        ```java
        void createEntry(int hash, K key, V value, int bucketIndex) {
            Entry<K,V> e = table[bucketIndex];
            // 将新的Enrty元素插入到对应桶的表头
            table[bucketIndex] = new Entry<>(hash, key, value, e);
            size++;
        }
        ```
        Entry<>实例化的源码如下：
        ```java
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n; // 将原先桶的表头向后移动
            key = k;
            hash = h;
        }
        ```
        在整个插入操作中，有一个很重要的操作，便是对table数组扩容，扩容的算法相对简单，但是在多线程下它却容易引发一个线程安全的问题。

        注：扩容需要会把原先table中的值移动到新的数组中，再赋值给table变量，一个合适的初始大小和负载因子能够提高效率。

    4. 线程不安全

        在多线程环境下，假设有容器map，其存储的情况如下图所示（淡蓝色为已有数据）。
        ![](ali/ali-java-hashMap-data.jpg)

        此时的map已经达到了扩容阈值12（16 * 0.75 = 12），而此时线程A与线程B同时对map容器进行插入操作，那么都需要扩容。此时可能出现的情况如下：线程A与线程B都进行了扩容，此时便有两个新的table，那么再赋值给原先的table变量时，便会出现其中一个newTable会被覆盖，假如线程B扩容的newTable覆盖了线程A扩容的newTable，并且是在A已经执行了插入操作之后，那么就会出现线程A的插入失效问题，也即是如下图中的两个table只能有一个会最后存在，而其中一个插入的值会被舍弃的问题。
        ![](ali/ali-java-hashMap-expand.jpg)

        这便是HashMap的线程不安全性，当然这只是其中的一点。而要消除这种隐患，则可以加锁或使用HashTable和ConcurrentHashMap这样的线程安全类，但是HashTable不被建议使用，推荐使用ConcurrentHashMap容器。

    原文：https://blog.csdn.net/t894690230/article/details/51323946

10. JVM内存结构，为什么需要GC？ 
    #### 一、JVM结构
    根据《java虚拟机规范》规定，JVM的基本结构一般如下图所示：

    ![](ali/ali-java-jvm-structure1.jpg)    ![](ali/ali-java-jvm-structure2.jpg)


    从左图可知，JVM主要包括四个部分：

    1. 类加载器（ClassLoader）:在JVM启动时或者在类运行时将需要的class加载到JVM中。（右图表示了从java源文件到JVM的整个过程，可配合理解。 关于类的加载机制，可以参考http://blog.csdn.net/tonytfjing/article/details/47212291）

    2. 执行引擎：负责执行class文件中包含的字节码指令（执行引擎的工作机制，这里也不细说了，这里主要介绍JVM结构）；

    3. 内存区（也叫运行时数据区）：是在JVM运行的时候操作所分配的内存区。运行时内存区主要可以划分为5个区域，如图：

        ![](ali/ali-java-jvm-mem.jpg)

        * 方法区(Method Area)：用于存储类结构信息的地方，包括常量池、静态变量、构造函数等。虽然JVM规范把方法区描述为堆的一个逻辑部分， 但它却有个别名non-heap（非堆），所以大家不要搞混淆了。方法区还包含一个运行时常量池。
        * java堆(Heap)：存储java实例或者对象的地方。这块是GC的主要区域（后面解释）。从存储的内容我们可以很容易知道，方法区和堆是被所有java线程共享的。
        * java栈(Stack)：java栈总是和线程关联在一起，每当创建一个线程时，JVM就会为这个线程创建一个对应的java栈。在这个java栈中又会包含多个栈帧，每运行一个方法就创建一个栈帧，用于存储局部变量表、操作栈、方法返回值等。每一个方法从调用直至执行完成的过程，就对应一个栈帧在java栈中入栈到出栈的过程。所以java栈是现成私有的。
        * 程序计数器(PC Register)：用于保存当前线程执行的内存地址。由于JVM程序是多线程执行的（线程轮流切换），所以为了保证线程切换回来后，还能恢复到原先状态，就需要一个独立的计数器，记录之前中断的地方，可见程序计数器也是线程私有的。
        * 本地方法栈(Native Method Stack)：和java栈的作用差不多，只不过是为JVM使用到的native方法服务的。
    4. 本地方法接口：主要是调用C或C++实现的本地方法及返回结果。
    
    #### 二、内存分配
    我觉得了解垃圾回收之前，得先了解JVM是怎么分配内存的，然后识别哪些内存是垃圾需要回收，最后才是用什么方式回收。

    Java的内存分配原理与C/C++不同，C/C++每次申请内存时都要malloc进行系统调用，而系统调用发生在内核空间，每次都要中断进行切换，这需要一定的开销，而Java虚拟机是先一次性分配一块较大的空间，然后每次new时都在该空间上进行分配和释放，减少了系统调用的次数，节省了一定的开销，这有点类似于内存池的概念；二是有了这块空间过后，如何进行分配和回收就跟GC机制有关了。

    java一般内存申请有两种：**静态内存和动态内存**。很容易理解，编译时就能够确定的内存就是静态内存，即内存是固定的，系统一次性分配，比如int类型变量；动态内存分配就是在程序执行时才知道要分配的存储空间大小，比如java对象的内存空间。根据上面我们知道，java栈、程序计数器、本地方法栈都是线程私有的，线程生就生，线程灭就灭，栈中的栈帧随着方法的结束也会撤销，内存自然就跟着回收了。所以这几个区域的内存分配与回收是确定的，我们不需要管的。但是java堆和方法区则不一样，我们只有在程序运行期间才知道会创建哪些对象，所以这部分内存的分配和回收都是动态的。一般我们所说的垃圾回收也是针对的这一部分。

    总之Stack的内存管理是顺序分配的，而且定长，不存在内存回收问题；而Heap 则是为java对象的实例随机分配内存，不定长度，所以存在内存分配和回收的问题；

    #### 三、垃圾检测、回收算法
    垃圾收集器一般必须完成两件事：检测出垃圾；回收垃圾。怎么检测出垃圾？一般有以下几种方法：

    引用计数法：给一个对象添加引用计数器，每当有个地方引用它，计数器就加1；引用失效就减1。

    好了，问题来了，如果我有两个对象A和B，互相引用，除此之外，没有其他任何对象引用它们，实际上这两个对象已经无法访问，即是我们说的垃圾对象。但是互相引用，计数不为0，导致无法回收，所以还有另一种方法：

    可达性分析算法：以根集对象为起始点进行搜索，如果有对象不可达的话，即是垃圾对象。这里的根集一般包括java栈中引用的对象、方法区常良池中引用的对象

    本地方法中引用的对象等。

    总之，JVM在做垃圾回收的时候，会检查堆中的所有对象是否会被这些根集对象引用，不能够被引用的对象就会被垃圾收集器回收。一般回收算法也有如下几种：

    1. 标记-清除（Mark-sweep）

    算法和名字一样，分为两个阶段：标记和清除。标记所有需要回收的对象，然后统一回收。这是最基础的算法，后续的收集算法都是基于这个算法扩展的。

    不足：效率低；标记清除之后会产生大量碎片。效果图如下：

    ![](ali/ali-java-jvm-mark-sweep.png)

    2. 复制（Copying）

    此算法把内存空间划为两个相等的区域，每次只使用其中一个区域。垃圾回收时，遍历当前使用区域，把正在使用中的对象复制到另外一个区域中。此算法每次只处理正在使用中的对象，因此复制成本比较小，同时复制过去以后还能进行相应的内存整理，不会出现“碎片”问题。当然，此算法的缺点也是很明显的，就是需要两倍内存空间。效果图如下：

    ![](ali/ali-java-jvm-copy.png)

    3. 标记-整理（Mark-Compact）

    此算法结合了“标记-清除”和“复制”两个算法的优点。也是分两阶段，第一阶段从根节点开始标记所有被引用对象，第二阶段遍历整个堆，把清除未标记对象并且把存活对象“压缩”到堆的其中一块，按顺序排放。此算法避免了“标记-清除”的碎片问题，同时也避免了“复制”算法的空间问题。效果图如下：

    ![](ali/ali-java-jvm-mark-compact.png)

    （1,2,3 图文摘自 http://pengjiaheng.iteye.com/blog/520228，感谢原作者。）

    4. 分代收集算法

    这是当前商业虚拟机常用的垃圾收集算法。分代的垃圾回收策略，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，不同生命周期的对象可以采取不同的收集方式，以便提高回收效率。

    **为什么要运用分代垃圾回收策略？**在java程序运行的过程中，会产生大量的对象，因每个对象所能承担的职责不同所具有的功能不同所以也有着不一样的生命周期，有的对象生命周期较长，比如Http请求中的Session对象，线程，Socket连接等；有的对象生命周期较短，比如String对象，由于其不变类的特性，有的在使用一次后即可回收。试想，在不进行对象存活时间区分的情况下，每次垃圾回收都是对整个堆空间进行回收，那么消耗的时间相对会很长，而且对于存活时间较长的对象进行的扫描工作等都是徒劳。因此就需要引入分治的思想，所谓分治的思想就是因地制宜，将对象进行代的划分，把不同生命周期的对象放在不同的代上使用不同的垃圾回收方式。

    **如何划分？**将对象按其生命周期的不同划分成：年轻代(Young Generation)、年老代(Old Generation)、持久代(Permanent Generation)。其中持久代主要存放的是类信息，所以与java对象的回收关系不大，与回收息息相关的是年轻代和年老代。这里有个比喻很形象

    “假设你是一个普通的 Java 对象，你出生在 Eden 区，在 Eden 区有许多和你差不多的小兄弟、小姐妹，可以把 Eden 区当成幼儿园，在这个幼儿园里大家玩了很长时间。Eden 区不能无休止地放你们在里面，所以当年纪稍大，你就要被送到学校去上学，这里假设从小学到高中都称为 Survivor 区。开始的时候你在 Survivor 区里面划分出来的的“From”区，读到高年级了，就进了 Survivor 区的“To”区，中间由于学习成绩不稳定，还经常来回折腾。直到你 18 岁的时候，高中毕业了，该去社会上闯闯了。于是你就去了年老代，年老代里面人也很多。在年老代里，你生活了 20 年 (每次 GC 加一岁)，最后寿终正寝，被 GC 回收。有一点没有提，你在年老代遇到了一个同学，他的名字叫爱德华 (慕光之城里的帅哥吸血鬼)，他以及他的家族永远不会死，那么他们就生活在永生代。”

    具体区域可以通过VisualVM中的VisaulGC插件查看，如图（openjdk 1.7）：
    ![](ali/ali-java-jvm-visualvm.png)

    年轻代：是所有新对象产生的地方。年轻代被分为3个部分——Enden区和两个Survivor区（From和to）当Eden区被对象填满时，就会执行Minor GC。并把所有存活下来的对象转移到其中一个survivor区（假设为from区）。Minor GC同样会检查存活下来的对象，并把它们转移到另一个survivor区（假设为to区）。这样在一段时间内，总会有一个空的survivor区。经过多次GC周期后，仍然存活下来的对象会被转移到年老代内存空间。通常这是在年轻代有资格提升到年老代前通过设定年龄阈值来完成的。需要注意，Survivor的两个区是对称的，没先后关系，from和to是相对的。

    年老代：在年轻代中经历了N次回收后仍然没有被清除的对象，就会被放到年老代中，可以说他们都是久经沙场而不亡的一代，都是生命周期较长的对象。对于年老代和永久代，就不能再采用像年轻代中那样搬移腾挪的回收算法，因为那些对于这些回收战场上的老兵来说是小儿科。通常会在老年代内存被占满时将会触发Full GC,回收整个堆内存。

    持久代：用于存放静态文件，比如java类、方法等。持久代对垃圾回收没有显著的影响。 

    分代回收的效果图如下：
    ![](ali/ali-java-jvm-persist.png)

    我这里之所以最后讲分代，是因为分代里涉及了前面几种算法。年轻代：涉及了复制算法；年老代：涉及了“标记-整理（Mark-Sweep）”的算法。

    四、垃圾收集器
    垃圾收集算法是内存回收的方法论，而实现这些方法论的则是垃圾收集器。不同厂商不同版本JVM所提供的垃圾收集器可能不同，这里参照《深入理解Java虚拟机》说的是JDK1.7版本Hotspot虚拟机，关于垃圾收集器有篇博文总结的不错，我就不说了，详见：
    原文：https://blog.csdn.net/tonytfjing/article/details/44278233 

11. NIO模型，select/epoll的区别，多路复用的原理 

    select，poll，epoll都是IO多路复用的机制。I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。关于这三种IO多路复用的用法，前面三篇总结写的很清楚，并用服务器回射echo程序进行了测试。连接如下所示：

    select：http://www.cnblogs.com/Anker/archive/2013/08/14/3258674.html

    poll：http://www.cnblogs.com/Anker/archive/2013/08/15/3261006.html

    epoll：http://www.cnblogs.com/Anker/archive/2013/08/17/3263780.html

    今天对这三种IO多路复用进行对比，参考网上和书上面的资料，整理如下：

    #### select实现
    select的调用过程如下所示：
    ![](ali/ali-java-io-select.png)

    1. 使用copy_from_user从用户空间拷贝fd_set到内核空间
    2. 注册回调函数__pollwait
    3. 遍历所有fd，调用其对应的poll方法（对于socket，这个poll方法是sock_poll，sock_poll根据情况会调用到tcp_poll,udp_poll或者datagram_poll）
    4. 以tcp_poll为例，其核心实现就是__pollwait，也就是上面注册的回调函数。
    5. __pollwait的主要工作就是把current（当前进程）挂到设备的等待队列中，不同的设备有不同的等待队列，对于tcp_poll来说，其等待队列是sk->sk_sleep（注意把进程挂到等待队列中并不代表进程已经睡眠了）。在设备收到一条消息（网络设备）或填写完文件数据（磁盘设备）后，会唤醒设备等待队列上睡眠的进程，这时current便被唤醒了。
    6. poll方法返回时会返回一个描述读写操作是否就绪的mask掩码，根据这个mask掩码给fd_set赋值。
    7. 如果遍历完所有的fd，还没有返回一个可读写的mask掩码，则会调用schedule_timeout是调用select的进程（也就是current）进入睡眠。当设备驱动发生自身资源可读写后，会唤醒其等待队列上睡眠的进程。如果超过一定的超时时间（schedule_timeout指定），还是没人唤醒，则调用select的进程会重新被唤醒获得CPU，进而重新遍历fd，判断有没有就绪的fd。
    8. 把fd_set从内核空间拷贝到用户空间。

    总结：

    select的几大缺点：

    1. 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
    2. 同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大
    3. select支持的文件描述符数量太小了，默认是1024

    #### poll实现

    poll的实现和select非常相似，只是描述fd集合的方式不同，poll使用pollfd结构而不是select的fd_set结构，其他的都差不多。

    关于select和poll的实现分析，可以参考下面几篇博文：

    http://blog.csdn.net/lizhiguo0532/article/details/6568964#comments

    http://blog.csdn.net/lizhiguo0532/article/details/6568968

    http://blog.csdn.net/lizhiguo0532/article/details/6568969

    http://www.ibm.com/developerworks/cn/linux/l-cn-edntwk/index.html?ca=drs-

    http://linux.chinaunix.net/techdoc/net/2009/05/03/1109887.shtml

    #### epoll

    epoll既然是对select和poll的改进，就应该能避免上述的三个缺点。那epoll都是怎么解决的呢？在此之前，我们先看一下epoll和select和poll的调用接口上的不同，select和poll都只提供了一个函数——select或者poll函数。而epoll提供了三个函数，epoll_create,epoll_ctl和epoll_wait，epoll_create是创建一个epoll句柄；epoll_ctl是注册要监听的事件类型；epoll_wait则是等待事件的产生。

    对于第一个缺点，epoll的解决方案在epoll_ctl函数中。每次注册新的事件到epoll句柄中时（在epoll_ctl中指定EPOLL_CTL_ADD），会把所有的fd拷贝进内核，而不是在epoll_wait的时候重复拷贝。epoll保证了每个fd在整个过程中只会拷贝一次。

    对于第二个缺点，epoll的解决方案不像select或poll一样每次都把current轮流加入fd对应的设备等待队列中，而只在epoll_ctl时把current挂一遍（这一遍必不可少）并为每个fd指定一个回调函数，当设备就绪，唤醒等待队列上的等待者时，就会调用这个回调函数，而这个回调函数会把就绪的fd加入一个就绪链表）。epoll_wait的工作实际上就是在这个就绪链表中查看有没有就绪的fd（利用schedule_timeout()实现睡一会，判断一会的效果，和select实现中的第7步是类似的）。

    对于第三个缺点，epoll没有这个限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左右，具体数目可以cat /proc/sys/fs/file-max察看,一般来说这个数目和系统内存关系很大。

    总结：

    1. select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用epoll_wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在epoll_wait中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间。这就是回调机制带来的性能提升。

    2. select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次，而epoll只要一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内部定义的等待队列）。这也能节省不少的开销。

    参考资料：

    http://www.cnblogs.com/apprentice89/archive/2013/05/09/3070051.html

    http://www.linuxidc.com/Linux/2012-05/59873p3.htm

    http://xingyunbaijunwei.blog.163.com/blog/static/76538067201241685556302/

    http://blog.csdn.net/kkxgx/article/details/7717125

    https://banu.com/blog/2/how-to-use-epoll-a-complete-example-in-c/epoll-example.c

    原文：https://www.cnblogs.com/Anker/p/3265058.html

12. Java中一个字符占多少个字节，扩展再问int, long, double占多少字节 
    1. 字节：byte：用来计量存储容量的一种计量单位；位：bit
    2. 一个字节等于8位  1byte = 8bit
        * 整型：
            * byte:1个字节 8位 -128~127
            * short ：2个字节 16位
            * int ：4个字节 32位
            * long：8个字节 64位
        * 浮点型：
            * float：4个字节 32 位
            * double ：8个字节 64位

            注：默认的是double类型，如3.14是double类型的，加后缀F（3.14F）则为float类型的。
        * char类型：

            char：2个字节。所以一个char类型的可以存储一个汉字。
        * Boolean 类型

            boolean: （true or false）（并未指明是多少字节  1字节  1位 4字节）

    补充：BigInteger类实现了任意精度的整数运算，BigDecimal实现了任意精度的浮点数运算。

    原文：https://blog.csdn.net/m0_37479246/article/details/79492828

13. 创建一个类的实例都有哪些办法？ 

    1. 关键字 new。工厂模式是对这种方式的包装；
    2. 类实现克隆接口，克隆一个实例。原型模式是一个应用实例；
    3. 用该类的加载器，newinstance。java的反射，反射使用实例：Spring的依赖注入、切面编程中动态代理
    4. sun.misc.Unsafe类，allocateInstance方法创建一个实例。（Java官方也不建议直接使用的Unsafe类，据说Oracle正在计划从Java 9中去掉Unsafe类）
    5. 实现序列化接口的类，通过IO流反序列化读取一个类，获得实例。

    原文：https://blog.csdn.net/lizhen54/article/details/74544801 

14. final/finally/finalize的区别？ 
    * final

        用于修饰类、成员变量和成员方法。
        * final修饰的类，不能被继承（String、StrngBuilder、StringBuffer、Math，不可变类），其中所有的方法都不能被重写，所以不能同时用abstract和final修饰（abstract修饰的是抽象类，抽象类是用于被子类继承的，和final起相反的作用）；
        * final修饰的方法不能被重写，但是子类可以用父类中final修饰的方法；
        * final修饰的成员变量是不可变的，如果成员变量是基本数据类型，初始化之后成员变量的值不能被改变，如果成员变量是引用类型，那么它只能指向初始化时指向的那个对象，不能再指向别的对象，但是对象中的内容是允许改变的。

    * finally

        finally是在异常处理时提供finally块来执行任何清除操作。不管有没有异常被抛出、捕获都会被执行。
        * try块中的内容是在无异常时执行到结束。
        * catch块中的内容，是在try块内容发生catch所声明的异常时，跳转到catch块中执行。
        * finally块则是无论异常是否发生都会执行finally块的内容，所以在代码逻辑中有需要无论发生什么都必须执行的代码，可以放在finally块中。

        try…catch…finally结构的语法格式
        ```java
        try{
            //可能会出现异常的代码段
        }catch(异常类名 处理该异常对象){
            //异常处理代码段
        }finally{
            //总是需要执行的代码段
        }
        ```
    * finalize

        finalize是方法名，java技术允许使用finalize()方法在垃圾收集器将对象从内存中清楚出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的，它是在Object类中定义的，因此所有的类都继承了它。子类覆盖finalize()方法以整理系统资源或者执行其他清理工作。finalize()方法是在垃圾收集器删除对象之前对这个对象调用的。

    原文：https://blog.csdn.net/qq_41939384/article/details/80662933

15. Session/Cookie的区别？ 
    #### 前言
    
    HTTP是一种无状态的协议，为了分辨链接是谁发起的，需自己去解决这个问题。不然有些情况下即使是同一个网站每打开一个页面也都要登录一下。而Session和Cookie就是为解决这个问题而提出来的两个机制。
    
    #### 应用场景
    * 登录网站，今输入用户名密码登录了，第二天再打开很多情况下就直接打开了。这个时候用到的一个机制就是cookie。
    * session一个场景是购物车，添加了商品之后客户端处可以知道添加了哪些商品，而服务器端如何判别呢，所以也需要存储一些信息就用到了session。

    #### 两者说明与比较
    1. Cookie

        通俗讲，是访问某些网站后在本地存储的一些网站相关信息，下次访问时减少一些步骤。更准确的说法是：Cookies是服务器在本地机器上存储的小段文本并随每一个请求发送至同一服务器，是在客户端保持状态的方案。
        Cookie的主要内容包括：名字，值，过期时间，路径和域。使用Fiddler抓包就可以看见，比方说我们打开百度的某个网站可以看到Headers包括Cookie，如下： 
        ```java
        BIDUPSID: 9D2194F1CB8D1E56272947F6B0E5D47E 
        PSTM: 1472480791 
        BAIDUID: 3C64D3C3F1753134D13C33AFD2B38367:FG 
        ispeed_lsm: 2 
        MCITY: -131: 
        pgv_pvi: 3797581824 
        pgv_si: s9468756992 
        BDUSS: JhNXVoQmhPYTVENEdIUnQ5S05xcHZMMVY5QzFRNVh5SzZoV0xMVDR6RzV-bEJZSVFBQUFBJCQAAAAAAAAAAAEAAACteXsbYnRfY2hpbGQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAALlxKVi5cSlYZj 
        BD_HOME: 1 
        H_PS_PSSID: 1423_21080_17001_21454_21408_21530_21377_21525_21193_21340 
        BD_UPN: 123253 
        sug: 3 
        sugstore: 0 
        ORIGIN: 0 
        bdime: 0
        ```
        key, value形式。过期时间可设置的，
        * 不设，存储在内存当中，则浏览器关掉就消失了，称为Session cookie/transient cookie
        * 按设置的时间来存储在硬盘上的，过期后自动清除，Persistent cookie/permenent cookie。

        路径和域就是对应的域名，a网站的cookie自然不能给b用。

    2. Session
        * 存在服务器的一种用来存放用户数据的类HashTable结构。
        * 浏览器第一次发送请求时，服务器自动生成了一HashTable和一Session ID来唯一标识这个HashTable，并将其通过响应发送到浏览器。浏览器第二次发送请求会将前一次服务器响应中的Session ID放在请求中一并发送到服务器上，服务器从请求中提取出Session ID，并和保存的所有Session ID进行对比，找到这个用户对应的HashTable。 
            * 一般这个值会有个时间限制，超时后毁掉这个值，默认30分钟。
        * 当用户在应用程序的 Web页间跳转时，存储在 Session 对象中的变量不会丢失而是在整个用户会话中一直存在下去。
        * Session的实现方式和Cookie有一定关系。建立一个连接就生成一个session id，打开几个页面就好几个了，这里就用到了Cookie，把session id存在Cookie中，每次访问的时候将Session id带过去就可以识别了.

    3. 区别
        * 存储数据量方面：session 能够存储任意的 java 对象，cookie 只能存储 String 类型的对象
        * session在服务端，cookie在客户端。因Cookie在客户端所以可以编辑伪造，不是十分安全。
        * Session过多时会消耗服务器资源，大型网站会有专门Session服务器，Cookie存在客户端没问题。
        * 域的支持范围不一样，比方说a.com的Cookie在a.com下都能用，而www.a.com的Session在api.a.com下都不能用，解决这个问题的办法是JSONP或者跨域资源共享。

    #### session多服务器间共享

    1. 服务器实现的 session 复制或 session 共享，如 webSphere或 JBOSS 在搭集群时配置实现 session 复制或 session 共享.致命缺点:不好扩展和移植。
    2. 利用成熟技术做session复制，如12306使用的gemfire，如常见内存数据库redis或memorycache，虽较普适但依赖第三方.
    3. 将 session维护在客户端，利用 cookie，但客户端存在风险数据不安全，且可以存放的数据量较小，所以将session维护在客户端还要对 session 中的信息加密。
    4. 第二种方案和第三种方案的合体，可用gemfire实现 session 复制共享，还可将session 维护在 redis中实现 session 共享，同时可将 session 维护在客户端的cookie 中，但前提是数据要加密。

    这三种方式可迅速切换，而不影响应用正常执行。在实践中，首选 gemfire 或者 redis 作为 session 共享的载体，一旦 session 不稳定出现问题的时候，可以紧急切换 cookie 维护 session 作为备用，不影响应用提供服务

    #### 单点登录中，cookie 被禁用了怎么办？（一点登陆，子网站其他系统不用再登陆）
    单点登录的原理是后端生成一个 session ID，设置到 cookie，后面所有请求浏览器都会带上cookie，然后服务端从cookie获取 session ID，查询到用户信息。
    所以，保持登录的关键不是cookie，而是通过cookie 保存和传输的 session ID，本质是能获取用户信息的数据。
    除了cookie，还常用 HTTP 请求头来传输。但这个请求头浏览器不会像cookie一样自动携带，需手工处理

    原文：https://blog.csdn.net/liyifan687/article/details/80077928

16. String/StringBuffer/StringBuilder的区别，扩展再问他们的实现？ 
    1. 三者在执行速度方面的比较：StringBuilder >  StringBuffer  >  String

    2. String <（StringBuffer，StringBuilder）的原因

        * String：字符串常
        * StringBuffer：字符创变量
        * StringBuilder：字符创变量
        ```java
        String s = "abcd";
        s = s+1;
        System.out.print(s);// result : abcd1
        ```
        首先创建对象s，赋予一个abcd，然后再创建一个新的对象s用来执行第二行代码，也就是说我们之前对象s并没有变化，所以我们说String类型是不可改变的对象了，由于这种机制，每当用String操作字符串时，实际上是在不断的创建新的对象，而原来的对象就会变为垃圾被GC回收掉，可想而知这样执行效率会有多低。

        而StringBuffer与StringBuilder就不一样了，他们是字符串变量，是可改变的对象，每当我们用它们对字符串做操作时，实际上是在一个对象上操作的，这样就不会像String一样创建一些而外的对象进行操作了，当然速度就快了

    3. 一个特殊的例子：
        ```java
        String str = "This is only a" + " simple" + " test"; //等同于 String str = "This is only a simple test";
        StringBuffer builder = new StringBuilder("This is only a").append(" simple").append(" test");
        ```
        这个时候StringBuffer居然速度上根本一点都不占优势。

    4. StringBuilder与 StringBuffer

        * StringBuilder：线程非安全的
        * StringBuffer：线程安全的

        当我们在字符串缓冲去被多个线程使用是，JVM不能保证StringBuilder的操作是安全的，虽然他的速度最快，但是可以保证StringBuffer是可以正确操作的。当然大多数情况下就是我们是在单线程下进行的操作，所以大多数情况下是建议用StringBuilder而不用StringBuffer的，就是速度的原因。

        对于三者使用的总结： 
        1. 如果要操作少量的数据用 = String
        2. 单线程操作字符串缓冲区 下操作大量数据 = StringBuilder
        3. 多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

    原文：https://www.cnblogs.com/strugglion/p/6389236.html
17. Servlet的生命周期？ 
    #### Servlet 生命周期
    1. Servlet 生命周期：Servlet 加载--->实例化--->服务--->销毁。
    2. init()：在Servlet的生命周期中，仅执行一次init()方法。它是在服务器装入Servlet时执行的，负责初始化Servlet对象。可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。无论有多少客户机访问Servlet，都不会重复执行init()。
    3. service()：它是Servlet的核心，负责响应客户的请求。每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”（ServletResponse）对象作为参数。在HttpServlet中已存在Service()方法。默认的服务功能是调用与HTTP请求的方法相应的do功能。
    4. destroy()： 仅执行一次，在服务器端停止且卸载Servlet时执行该方法。当Servlet对象退出生命周期时，负责释放占用的资源。一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确认在调用destroy()方法时，这些线程已经终止或完成。

    #### Tomcat 与 Servlet 是如何工作的：

    ![](ali/ali-java-servlet.jpg)

    步骤：
    1. Web Client 向Servlet容器（Tomcat）发出Http请求
    2. Servlet容器接收Web Client的请求
    3. Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中。
    4. Servlet容器创建一个HttpResponse对象
    5. Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给 HttpServlet 对象。
    6. HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息。
    7. HttpServlet调用HttpResponse对象的有关方法，生成响应数据。
    8. Servlet容器把HttpServlet的响应结果传给Web Client。

    #### Servlet工作原理：

    1. 首先简单解释一下Servlet接收和响应客户请求的过程，首先客户发送一个请求，Servlet是调用service()方法对请求进行响应的，通过源代码可见，service()方法中对请求的方式进行了匹配，选择调用doGet,doPost等这些方法，然后再进入对应的方法中调用逻辑层的方法，实现对客户的响应。在Servlet接口和GenericServlet中是没有doGet()、doPost()等等这些方法的，HttpServlet中定义了这些方法，但是都是返回error信息，所以，我们每次定义一个Servlet的时候，都必须实现doGet或doPost等这些方法。

    2. 每一个自定义的Servlet都必须实现Servlet的接口，Servlet接口中定义了五个方法，其中比较重要的三个方法涉及到Servlet的生命周期，分别是上文提到的init(),service(),destroy()方法。GenericServlet是一个通用的，不特定于任何协议的Servlet,它实现了Servlet接口。而HttpServlet继承于GenericServlet，因此HttpServlet也实现了Servlet接口。所以我们定义Servlet的时候只需要继承HttpServlet即可。

    3. Servlet接口和GenericServlet是不特定于任何协议的，而HttpServlet是特定于HTTP协议的类，所以HttpServlet中实现了service()方法，并将请求ServletRequest、ServletResponse 强转为HttpRequest 和 HttpResponse。

    #### 创建Servlet对象的时机

    1. Servlet容器启动时：读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法。
    2. 在Servlet容器启动后：客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet 对象的service方法。
    3. Servlet容器在启动时自动创建Servlet，这是由在web.xml文件中为Servlet设置的<load-on-startup>属性决定的。从中我们也能看到同一个类型的Servlet对象在Servlet容器中以单例的形式存在。
    ```xml
    <servlet>
            <servlet-name>Init</servlet-name>
            <servlet-class>org.xl.servlet.InitServlet</servlet-class>
            <load-on-startup>1</load-on-startup>
    </servlet>
    ```
    原文：http://www.cnblogs.com/xuekyo/archive/2013/02/24/2924072.html

18. 如何用Java分配一段连续的1G的内存空间？需要注意些什么？ 
    
    ByteBuffer.allocateDirect(1024*1024*1024);

19. Java有自己的内存回收机制，但为什么还存在内存泄露的问题呢？ 
    1. 我们需要知道 GC 在什么时候回收内存对象，什么样的内存对象会被 GC 认为是“不再使用”的。

        Java中对内存对象的访问，使用的是引用的方式。在 Java 代码中我们维护一个内存对象的引用变量，通过这个引用变量的值，我们可以访问到对应的内存地址中的内存对象空间。在 Java 程序中，这个引用变量本身既可以存放堆内存中，又可以放在代码栈的内存中（与基本数据类型相同）。 GC 线程会从代码栈中的引用变量开始跟踪，从而判定哪些内存是正在使用的。如果 GC 线程通过这种方式，无法跟踪到某一块堆内存，那么 GC 就认为这块内存将不再使用了（因为代码中已经无法访问这块内存了）。

        通过这种有向图的内存管理方式，当一个内存对象失去了所有的引用之后，GC 就可以将其回收。反过来说，如果这个对象还存在引用，那么它将不会被 GC 回收，哪怕是 Java 虚拟机抛出 OutOfMemoryError 。

    2. java内存泄漏的根本原因是？

        内存对象明明已经不需要的时候，还仍然保留着这块内存和它的访问方式（引用）。

    3. 堆溢出:静态集合类中存储对象

        java.lang.OutOfMemoryError: Java heap space

    4. 栈溢出

        java.lang.StackOverflowError

    5. 代码栈是什么？
        ```java
        Vector v = new  Vector( 10 );  
        for  ( int  i = 1 ;i < 100 ; i ++ ){  
            Object o = new  Object();  
            v.add(o);  
            o = null ;  
        }
        ```
        在这个例子中，代码栈中存在Vector 对象的引用 v 和 Object 对象的引用 o 。
        在 For 循环中，我们不断的生成新的对象，然后将其添加到 Vector 对象中，之后将 o 引用置空。问题是当 o 引用被置空后，如果发生 GC ，我们创建的 Object 对象是否能够被 GC 回收呢？

        答案是否定的。

        因为， GC 在跟踪代码栈中的引用时，会发现 v 引用，而继续往下跟踪，就会发现 v 引用指向的内存空间中又存在指向 Object 对象的引用。也就是说尽管 o 引用已经被置空，但是 Object 对象仍然存在其他的引用，是可以被访问到的，所以 GC 无法将其释放掉。
        如果在此循环之后， Object 对象对程序已经没有任何作用，那么我们就认为此 Java 程序发生了内存泄漏。

    6. 内存泄漏在哪个领域比较常见？

        在移动设备对于内存和 CPU都有较严格的限制的情况下， Java 的内存溢出会导致程序效率低下、占用大量不需要的内存等问题。这将导致整个机器性能变差，严重的也会引起抛出 OutOfMemoryError ，导致程序崩溃。
    7. 如何避免内存泄漏？

        明确引用变量的生命周期，是方法内部的局部变量，还是类实例变量，与类实例生命周期相同的要声明为实例变量。

    要避免这种情况下的内存泄露，要求我们以C/C++ 的内存管理思维来管理自己分配的内存。

    第一，是在声明对象引用之前，明确内存对象的有效作用域。在一个函数内有效的内存对象，应该声明为 local 变量，与类实例生命周期相同的要声明为实例变量……以此类推。

    第二，在内存对象不再需要时，记得手动将其引用置空。

20. 什么是java序列化，如何实现java序列化?(写一个实例)？ 
    1. 串行化技术

        * Java 串行化技术可以使你将一个对象的状态写入一个Byte 流里，并且可以从其它地方把该Byte 流里的数据读出来，重新构造一个相同的对象。这种机制允许你将对象通过网络进行传播，并可以随时把对象持久化到数据库、文件等系统里。

            * Java的串行化机制是RMI、EJB等技术的技术基础
            * 用途：利用对象的串行化实现保存应用程序的当前工作状态，下次再启动的时候将自动地恢复到上次执行的状态。

        * 序列化就是一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化。可以对流化后的对象进行读写操作，也可将流化后的对象传输于网络之间。序列化是为了解决在对对象流进行读写操作时所引发的问题。

        * 序列化的实现：将需要被序列化的类实现Serializable接口，然后使用一个输出流(如：FileOutputStream)来构造一个ObjectOutputStream(对象流)对象，接着，使用ObjectOutputStream对象的writeObject(Object obj)方法就可以将参数为obj的对象写出(即保存其状态)，要恢复的话则用输入流。

    2. 串行化的特点

        1. 如果某个类能够被串行化，其子类也可以被串行化。如果该类有父类，则分两种情况来考虑，如果该父类已经实现了可串行化接口。则其父类的相应字段及属性的处理和该类相同；如果该类的父类没有实现可串行化接口，则该类的父类所有的字段属性将不会串行化。

        2. 声明为static和transient类型的成员数据不能被串行化。因为static代表类的状态， transient代表对象的临时数据；

        3. 相关的类和接口：在java.io包中提供的涉及对象的串行化的类与接口有ObjectOutput接口、ObjectOutputStream类、ObjectInput接口、ObjectInputStream类。

            * ObjectOutput接口：它继承DataOutput接口并且支持对象的串行化，其内的writeObject()方法实现存储一个对象。
            * ObjectInput接口：它继承DataInput接口并且支持对象的串行化，其内的readObject()方法实现读取一个对象。
            * ObjectOutputStream类：它继承OutputStream类并且实现ObjectOutput接口。利用该类来实现将对象存储（调用ObjectOutput接口中的writeObject()方法）。
            * ObjectInputStream类：它继承InputStream类并且实现ObjectInput接口。利用该类来实现读取一个对象（调用ObjectInput接口中的readObject()方法）。

    对于父类的处理，如果父类没有实现串行化接口，则其必须有默认的构造函数（即没有参数的构造函数）。否则编译的时候就会报错。在反串行化的时候，默认构造函数会被调用。但是若把父类标记为可以串行化，则在反串行化的时候，其默认构造函数不会被调用。这是为什么呢？这是因为Java 对串行化的对象进行反串行化的时候，直接从流里获取其对象数据来生成一个对象实例，而不是通过其构造函数来完成。
    ```java
    import java.io.*;

    public class Cat implements Serializable {
        private String name;
        public Cat () {
            this.name = "new cat";
        }
        public String getName() {
            return this.name;
        }
        public void setName(String name) {
            this.name = name;
        }

        public static void main(String[] args) {
            Cat cat = new Cat();
            try {
                FileOutputStream fos = new FileOutputStream("catDemo.out");
                ObjectOutputStream oos = new ObjectOutputStream(fos);
                System.out.println(" 1> " + cat.getName());
                cat.setName("My Cat");
                oos.writeObject(cat);
                oos.close();
            } catch (Exception ex) {  
                ex.printStackTrace();
            }
            try {
                FileInputStream fis = new FileInputStream("catDemo.out");
                ObjectInputStream ois = new ObjectInputStream(fis);
                cat = (Cat) ois.readObject();
                System.out.println(" 2> " + cat.getName());
                ois.close();
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }
    ```
    writeObject和readObject本身就是线程安全的，传输过程中是不允许被并发访问的。所以对象能一个一个接连不断的传过来
    原文：https://blog.csdn.net/cselmu9/article/details/41908741

21. String s = new String("abc");创建了几个 String Object? 

    对于String s=new String("abc"),这句话我们来大致分析一下步骤:
    1. 在虚拟机栈中为String类型的s分配内存
    2. 在堆中为分配一块内存用来保存"abc"
    3. 将堆中指向"abc"的地址赋值给s

    这样来分析的话是在堆中创建了一个对象,而有些人会认为s也是对象啊,这不是两个吗?其实不然,s只是一个对象的引用,他指向了在堆中创建的那个对象.

    原文：https://blog.csdn.net/qq_35634181/article/details/82961297 