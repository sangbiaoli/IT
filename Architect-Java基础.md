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

    参考：http://www.importnew.com/12399.html

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

    参考：https://blog.csdn.net/JetaimeHQ/article/details/83031899

3. 常用的集合类有哪些？比如List如何排序？ 

4. ArrayList和LinkedList内部的实现大致是怎样的？他们之间的区别和优缺点？ 

5. 内存溢出是怎么回事？请举一个例子？ 

6. ==和equals的区别？ 

7. hashCode方法的作用？ 

8. NIO是什么？适用于何种场景？ 

9. HashMap实现原理，如何保证HashMap的线程安全？ 

10. JVM内存结构，为什么需要GC？ 

11. NIO模型，select/epoll的区别，多路复用的原理 

12. Java中一个字符占多少个字节，扩展再问int, long, double占多少字节 

13. 创建一个类的实例都有哪些办法？ 

14. final/finally/finalize的区别？ 

15. Session/Cookie的区别？ 

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

    参考：https://www.cnblogs.com/strugglion/p/6389236.html
17. Servlet的生命周期？ 

18. 如何用Java分配一段连续的1G的内存空间？需要注意些什么？ 

19. Java有自己的内存回收机制，但为什么还存在内存泄露的问题呢？ 

20. 什么是java序列化，如何实现java序列化?(写一个实例)？ 

21. String s = new String("abc");创建了几个 String Object? 