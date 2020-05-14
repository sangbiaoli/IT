## JDK8新特性

* Lambda表达式
* 新的日期API
* 引入Optional
* 使用Base64
* 接口的默认方法和静态方法
* 新增方法引用格式
* 新增Stream类
* 注解相关的改变
* 支持并行（parallel）数组
* 对并发类（Concurrency）的扩展。

1. Lambda表达式

    Lambda 表达式也可称为闭包，是推动 Java 8 发布的最重要新特性。lambda表达式本质上是一个匿名方法。Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中）或者把代码看成数据。使用 Lambda 表达式可以使代码变的更加简洁紧凑。在最简单的形式中，一个lambda可以由：用逗号分隔的参数列表、–>符号、函数体三部分表示，在某些情况下lambda的函数体会更加复杂，这时可以把函数体放到在一对花括号中，就像在Java中定义普通函数一样。Lambda可以引用类的成员变量与局部变量（如果这些变量不是final的话，它们会被隐含的转为final，这样效率更高）。Lambda可能会返回一个值。返回值的类型也是由编译器推测出来的。如果lambda的函数体只有一行的话，那么没有必要显式使用return语句。

    如何使现有的函数友好地支持lambda。最终采取的方法是：增加函数式接口的概念。函数式接口就是接口里面必须有且只有一个抽象方法的普通接口，像这样的接口可以被隐式转换为lambda表达式成为函数式接口。 在可以使用lambda表达式的地方，方法声明时必须包含一个函数式的接口。 任何函数式接口都可以使用lambda表达式替换，例如：ActionListener、Comparator、Runnable。

    函数式接口是容易出错的：如有某个人在接口定义中增加了另一个方法，这时，这个接口就不再是函数式的了，并且编译过程也会失败。为了克服函数式接口的这种脆弱性并且能够明确声明接口作为函数式接口的意图，Java 8增加了一种特殊的注解@FunctionalInterface，但是默认方法与静态方法并不影响函数式接口的契约，可以任意使用。

     

    使用lambda表达式替换匿名类，而实现Runnable接口是匿名类的最好示例。通过() -> {}代码块替代了整个匿名类。

    java 8之前：
    
    ```java
    new Thread(new Runnable() {

        @Override

        public void run() {

        System.out.println("Before Java8, too much code for too little to do");

        }

    }).start();
    ```
 
    Java 8方式：

    ```java
    new Thread(() -> System.out.println("In Java8, Lambda expression rocks !!") ).start();
    ```

    Lambda 表达式免去了使用匿名方法的麻烦，并且给予Java简单但是强大的函数化的编程能力。

2. 新的日期API

    Java 8通过发布新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。在旧版的 Java 中，日期时间 API 存在诸多问题，比如：

    1. 非线程安全

        java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。

    2. 设计很差
    
        Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。

    3. 时区处理麻烦

        日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。
        
        Java 8 在 java.time 包下提供了很多新的 API。以下为两个比较重要的 API：

        1. Local(本地) − 简化了日期时间的处理，没有时区的问题。

        2. Zoned(时区) − 通过制定的时区处理日期时间。

    新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。

3. Optional

    Optional类实际上是个容器：它可以保存类型T的值，或者仅仅保存null。Optional 类的引入很好的解决空指针异常。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。尽量避免在程序中直接调用Optional对象的get()和isPresent()方法，避免使用Optional类型声明实体类的属性。

    1. Optional.of(T t) : 创建一个 Optional 实例

    2. Optional.empty() : 创建一个空的 Optional 实例

    3. Optional.ofNullable(T t):若 t 不为 null,创建 Optional 实例,否则创建空实例

    4. isPresent() : 判断是否包含值 orElse(T t) : 如果调用对象包含值，返回该值，否则返回t

    5. orElseGet(Supplier s) :如果调用对象包含值，返回该值，否则返回 s 获取的值

    6. map(Function f): 如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty()

    7. flatMap(Function mapper):与 map 类似，要求返回值必须是Optional

    方法详细

    1. 创建optional对象，一般用ofNullable()而不用of()：

        1. empty() ：用于创建一个没有值的Optional对象：Optional<String> emptyOpt = Optional.empty();

        2. of() ：使用一个非空的值创建Optional对象：Optional<String> notNullOpt = Optional.of(str);

        3. ofNullable() ：接收一个可以为null的值：Optional<String> nullableOpt = Optional.ofNullable(str);

    2. 判断Null：

        1. isPresent()：如果创建的对象实例为非空值的话，isPresent()返回true，调用get()方法会返回该对象，如果没有值，调用isPresent()方法会返回false，调用get()方法抛出NullPointerException异常。

    3. 获取对象：

        1. get()

    4. 使用map提取对象的值，如果我们要获取User对象中的roleId属性值，常见的方式是先判断是否为null然后直接获取，但使用Optional中提供的map()方法可以以更简单的方式实现

     

    5. 使用orElse方法设置默认值，Optional类还包含其他方法用于获取值，这些方法分别为：

        1. orElse()：如果有值就返回，否则返回一个给定的值作为默认值；

        2. orElseGet()：与orElse()方法作用类似，区别在于生成默认值的方式不同。该方法接受一个Supplier<? extends T>函数式接口参数，用于生成默认值；

        3. orElseThrow()：与前面介绍的get()方法类似，当值为null时调用这两个方法都会抛出NullPointerException异常，区别在于该方法可以指定抛出的异常类型。

    6. 使用filter()方法过滤，filter()方法可用于判断Optional对象是否满足给定条件，一般用于条件过滤，在代码中，如果filter()方法中的Lambda表达式成立，filter()方法会返回当前Optional对象值，否则，返回一个值为空的Optional对象。

4. Base64

    Base64编码的作用 ：

    由于某些系统中只能使用ASCII字符。Base64就是用来将非ASCII字符的数据转换成ASCII字符的一种方法。 Base64其实不是安全领域下的加密解密算法，而是一种编码，也就是说，它是可以被翻译回原来的样子。它并不是一种加密过程。所以base64只能算是一个编码算法，对数据内容进行编码来适合传输。虽然base64编码过后原文也变成不能看到的字符格式，但是这种方式很初级，很简单。

    使用Base64编码原因 ：

    1. base64是网络上最常见的用于传输8bit字节代码的编码方式之一。有时我们需要把二进制数据编码为适合放在URL中的形式。这时采用base64编码具有不可读性，即所编码的数据不会被人直接看出。

    2. 用于在http环境下传递较长的标识信息。

    在Java 8中，Base64编码已经成为Java类库的标准，并内置了 Base64 编码的编码器和解码器。Base64工具类提供了一套静态方法获取下面三种BASE64编解码器：

    基本：输出被映射到一组字符A-Za-z0-9+/，编码不添加任何行标，输出的解码仅支持A-Za-z0-9+/。

    URL：输出映射到一组字符A-Za-z0-9+_，输出是URL和文件。

    MIME：输出隐射到MIME友好格式。输出每行不超过76字符，并且使用'\r'并跟随'\n'作为分割。编码输出最后没有行分割。

 

5. 接口的默认方法和静态方法

    Java 8用默认方法与静态方法这两个新概念来扩展接口的声明。默认方法与抽象方法不同之处在于抽象方法必须要求实现，但是默认方法则没有这个要求，就是接口可以有实现方法，而且不需要实现类去实现其方法。我们只需在方法名前面加个default关键字即可实现默认方法。为什么要有这个特性？以前当需要修改接口的时候，需要修改全部实现该接口的类。而引进的默认方法的目的是为了解决接口的修改与现有的实现不兼容的问题。

    默认方法语法格式如下：
    
    ```java
    public interface Vehicle {

       default void print(){

          System.out.println("我是一辆车!");

       }

    }
    ```

    当出现这样的情况，一个类实现了多个接口，且这些接口有相同的默认方法，这种情况的解决方法：

    1. 是创建自己的默认方法，来覆盖重写接口的默认方法

    2. 可以使用 super 来调用指定接口的默认方法

    Java 8 的另一个特性是接口可以声明（并且可以提供实现）静态方法。在JVM中，默认方法的实现是非常高效的，并且通过字节码指令为方法调用提供了支持。默认方法允许继续使用现有的Java接口，而同时能够保障正常的编译过程。尽管默认方法非常强大，但是在使用默认方法时我们需要小心注意一个地方：在声明一个默认方法前，请仔细思考是不是真的有必要使用默认方法，因为默认方法会带给程序歧义，并且在复杂的继承体系中容易产生编译错误。

6. 方法引用

    方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

    定义了4个方法的Car这个类作为例子，区分Java中支持的4种不同的方法引用。

    ```java
    public static class Car {

        public static Car create( final Supplier< Car > supplier ) {

            return supplier.get();

        }                       

        public static void collide( final Car car ) {

            System.out.println( "Collided " + car.toString() );

        }         

        public void follow( final Car another ) {

            System.out.println( "Following the " + another.toString() );

        }         

        public void repair() {  

            System.out.println( "Repaired " + this.toString() );

        }

    }
    ```

    第一种方法引用是构造器引用，它的语法是Class::new，或者更一般的Class< T >::new。请注意构造器没有参数。

    ```java
    final Car car = Car.create( Car::new );

    final List< Car > cars = Arrays.asList( car ); 
    ```

    第二种方法引用是静态方法引用，它的语法是Class::static_method。请注意这个方法接受一个Car类型的参数

    ```java
    cars.forEach( Car::collide );
    ```

    第三种方法引用是特定类的任意对象的方法引用，它的语法是Class::method。请注意，这个方法没有参数。

    ```java
    cars.forEach( Car::repair );
    ```

    第四种方法引用是特定对象的方法引用，它的语法是instance::method。请注意，这个方法接受一个Car类型的参数

    ```java
    final Car police = Car.create( Car::new );

    cars.forEach( police::follow );
    ```
 
7. Stream

    Java 8 API添加了一个新的抽象称为流Stream把真正的函数式编程风格引入到Java中，可以让你以一种声明的方式处理数据。Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。Stream API极大简化了集合框架的处理，这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

    Stream流有一些特性：

    1. Stream流不是一种数据结构，不保存数据，它只是在原数据集上定义了一组操作。

    2. 这些操作是惰性的，即每当访问到流中的一个元素，才会在此元素上执行这一系列操作。

    3. Stream不保存数据，故每个Stream流只能使用一次。

    所以这边有两个概念:流、管道。元素流在管道中经过中间操作的处理，最后由最终操作得到前面处理的结果。这里有2个操作：中间操作、最终操作。

    * 中间操作：返回结果都是Stream，故可以多个中间操作叠加。

    * 终止操作：用于返回我们最终需要的数据，只能有一个终止操作。

    使用Stream流，可以清楚地知道我们要对一个数据集做何种操作，可读性强。而且可以很轻松地获取并行化Stream流，不用自己编写多线程代码，可以更加专注于业务逻辑。默认情况下，从有序集合、生成器、迭代器产生的流或者通过调用Stream.sorted产生的流都是有序流，有序流在并行处理时会在处理完成之后恢复原顺序。无限流的存在，侧面说明了流是惰性的，即每当用到一个元素时，才会在这个元素上执行这一系列操作。

    使用Stream的基本步骤：

    1. 创建Stream

    2. 转换Stream，每次转换原有Stream对象不改变，返回一个新的Stream对象（可以有多次转换）

    3. 对Stream进行聚合操作，获取想要的结果



    #### 流的生成方法

    1. Collection接口的stream()或parallelStream()方法

    2. 静态的Stream.of()、Stream.empty()方法

    3. Arrays.stream(array, from, to)

    4. 静态的Stream.generate()方法生成无限流，接受一个不包含引元的函数

    5. 静态的Stream.iterate()方法生成无限流，接受一个种子值以及一个迭代函数

    6. Pattern接口的splitAsStream(input)方法

    7. 静态的Files.lines(path)、Files.lines(path, charSet)方法

    8. 静态的Stream.concat()方法将两个流连接起来

    #### 流的Intermediate方法(中间操作)

    1. filter(Predicate) ：将结果为false的元素过滤掉

    2. map(fun) ：转换元素的值，可以用方法引元或者lambda表达式

    3. flatMap(fun) ：若元素是流，将流摊平为正常元素，再进行元素转换

    4. limit(n) ：保留前n个元素

    5. skip(n) ：跳过前n个元素

    6. distinct() ：剔除重复元素

    7. sorted() ：将Comparable元素的流排序

    8. sorted(Comparator) ：将流元素按Comparator排序

    9. peek(fun) ：流不变，但会把每个元素传入fun执行，可以用作调试

    #### 流的Terminal方法(终结操作)

    1. 约简操作

        1. reduce(fun) ：从流中计算某个值，接受一个二元函数作为累积器，从前两个元素开始持续应用它，累积器的中间结果作为第一个参数，流元素作为第二个参数

        2. reduce(a, fun) ：a为幺元值，作为累积器的起点

        3. reduce(a, fun1, fun2) ：与二元变形类似，并发操作中，当累积器的第一个参数与第二个参数都为流元素类型时，可以对各个中间结果也应用累积器进行合并，但是当累积器的第一个参数不是流元素类型而是类型T的时候，各个中间结果也为类型T，需要fun2来将各个中间结果进行合并

    2. 收集操作

        1. iterator()：

        2. forEach(fun)：

        3. forEachOrdered(fun) ：可以应用在并行流上以保持元素顺序

        4. toArray()：

        5. toArray(T[] :: new) ：返回正确的元素类型

        6. collect(Collector)：

        7. collect(fun1, fun2, fun3) ：fun1转换流元素；fun2为累积器，将fun1的转换结果累积起来；fun3为组合器，将并行处理过程中累积器的各个结果组合起来

    3. 查找与收集操作

        1. max(Comparator)：返回流中最大值

        2. min(Comparator)：返回流中最小值

        3. count()：返回流中元素个数

        4. findFirst() ：返回第一个元素

        5. findAny() ：返回任意元素

        6. anyMatch(Predicate) ：任意元素匹配时返回true

        7. allMatch(Predicate) ：所有元素匹配时返回true

        8. noneMatch(Predicate) ：没有元素匹配时返回true

8. 注解相关

    1. 可以进行重复注解

        自从Java 5引入了注解机制，这一特性就变得非常流行并且广为使用。然而，使用注解的一个限制是相同的注解在同一位置只能声明一次，不能声明多次。Java 8打破了这条规则，引入了重复注解机制，这样相同的注解可以在同一地方声明多次。

        重复注解机制本身必须用@Repeatable注解。事实上，这并不是语言层面上的改变，更多的是编译器的技巧，底层的原理保持不变。

    2. 扩展注解的支持

        Java 8扩展了注解的上下文。现在几乎可以为任何东西添加注解：局部变量、泛型类、父类与接口的实现，就连方法的异常也能添加注解。

9. 并行（parallel）数组

    Java 8增加了大量的新方法来对数组进行并行处理。可以说，最重要的是parallelSort()方法，因为它可以在多核机器上极大提高数组排序的速度。下面的例子展示了新方法（parallelXxx）的使用。

    上面的代码片段使用了parallelSetAll()方法来对一个有20000个元素的数组进行随机赋值。然后，调用parallelSort方法。这个程序首先打印出前10个元素的值，之后对整个数组排序。这个程序在控制台上的输出如下（请注意数组元素是随机生产的）：

 

10. 并发（Concurrency）

    在新增Stream机制与lambda的基础之上，在java.util.concurrent.ConcurrentHashMap中加入了一些新方法来支持聚集操作。同时也在java.util.concurrent.ForkJoinPool类中加入了一些新方法来支持共有资源池（common pool）（请查看我们关于Java 并发的免费课程）。

    新增的java.util.concurrent.locks.StampedLock类提供一直基于容量的锁，这种锁有三个模型来控制读写操作（它被认为是不太有名的java.util.concurrent.locks.ReadWriteLock类的替代者）。

    在java.util.concurrent.atomic包中还增加了下面这些类：

    1. DoubleAccumulator

    2. DoubleAdder

    3. LongAccumulator

    4. LongAdder


原文：https://blog.csdn.net/ZytheMoon/article/details/89715618