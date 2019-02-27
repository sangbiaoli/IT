## DCL（Double Check Lock）单例模式详解

首先必须声明，在volatile出现之前，错误的DCL代码如下。在volatile出现之后，正确的DCL代码如下。代码如下：

```java
//错误的代码  
public class Singleton {  
    private static Singleton instance=null;  
    private Singleton(){}  
    public static Singleton getInstance(){  
        if(instance==null){  
            synchronized (Singleton.class) {  
                if(instance==null)  
                    instance=new Singleton();//mark行
            }
        }  
        return instance;  
    }  
}  
```

```java
//正确的代码  
public class Singleton {  
    private volatile static Singleton instance=null;//添加了volatile修饰符  
    private Singleton(){}  
    public static Singleton getInstance(){  
        if(instance==null){  
            synchronized (Singleton.class) {  
                if(instance==null)  
                    instance=new Singleton();
            }
        }  
        return instance;  
    }  
}  
```

我们来剖析错误的代码：

在错误的DCL代码中，代码行mark在JVM中有2种可能的执行顺序（JVM会对一些没有依赖要求的指令重排序。关于重排序，在下文中进行说明），分别为：

```C
//第一种执行顺序  
mem = allocate()  
mem.initial()  
instance = mem  
```

```C
//第二种执行顺序  
mem = allocate()  
instance = mem  
mem.initial()
```

假设现在有2个线程a和b，**线程a执行到mark行语句并按照第二种执行顺序执行完instance=mem指令。此时线程b执行getInstane方法会直接返回一个非空的instance，但是这个instance可能是未被完整创建的**。这个时候，DCL就出问题了。这就是大家对DCL口诛笔伐的原因。由此可见，错误的DCL的真正问题就在于：在没有同步的情况下读取一个共享变量，可能读到不完整的实例。也就是说，不在同步代码块中的if(instance==null)代码可能读取到不完整的instance实例。

为了解决类似DCL中出现的问题，JAVA在JMM中(JAVA内存模型)定义了一系列Happens-Before关系。如果两个操作之间缺少Happens-Before关系，那么JVM就可以对它们任意的重排序。比如volatile变量规则规定：对volatile变量的写入操作必须在对该变量的读操作之前进行。这就是DCL在使用volatile之后变得正确的原因。


原文：https://blog.csdn.net/chtnj/article/details/62222794
https://www.jianshu.com/p/0bf826fe9f07