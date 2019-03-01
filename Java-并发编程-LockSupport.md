## LockSupport

1. LockSupport功能简介
    1. 使用wait,notify阻塞唤醒线程
    2. 使用LockSupport阻塞唤醒线程
2. LockSupport的其他特色
    1. 可以先唤醒线程再阻塞线程
    2. 先唤醒线程两次再阻塞两次会发生什么
3. LockSupport阻塞和唤醒线程原理浅析
4. 总结

##

1. LockSupport功能简介

    在java并发包下各种同步组件的底层实现中，LockSupport的身影处处可见。JDK中的定义为用来创建锁和其他同步类的线程阻塞原语。

    ```
    *Basic thread blocking primitives for creating locks and other
    *synchronization classes.
    ```

    我们可以使用它来阻塞和唤醒线程,功能和wait,notify有些相似,但是LockSupport比起wait,notify功能更强大，也好用的多。

    1. 使用wait,notify阻塞唤醒线程

        ```java
        public class WaitNotifyTest {
            private static Object obj = new Object();

            public static void main(String[] args) {
                new Thread(new WaitThread()).start();
                new Thread(new NotifyThread()).start();
            }

            static class WaitThread implements Runnable {
                @Override
                public void run() {
                    synchronized (obj) {
                        System.out.println("start wait!");
                        try {
                            obj.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("end wait!");
                    }
                }
            }

            static class NotifyThread implements Runnable {
                @Override
                public void run() {
                    synchronized (obj) {
                        System.out.println("start notify!");
                        obj.notify();
                        System.out.println("end notify");
                    }
                }
            }
        }
        ```

        结果

        ```
        start wait!
        start notify!
        end notify
        end wait!
        ```

        使用wait，notify来实现等待唤醒功能至少有两个缺点：

        1. 由上面的例子可知,wait和notify都是Object中的方法,在调用这两个方法前必须先获得锁对象，这限制了其使用场合:只能在同步代码块中。
        2. 另一个缺点可能上面的例子不太明显，当对象的等待队列中有多个线程时，notify只能随机选择一个线程唤醒，无法唤醒指定的线程。

        而使用LockSupport的话，我们可以在任何场合使线程阻塞，同时也可以指定要唤醒的线程，相当的方便。

    2. 使用LockSupport阻塞唤醒线程

        ```java
        public class LockSupportTest {

            public static void main(String[] args) {
                Thread parkThread = new Thread(new ParkThread());
                parkThread.start();
                System.out.println("开始线程唤醒");
                LockSupport.unpark(parkThread);
                System.out.println("结束线程唤醒");

            }

            static class ParkThread implements Runnable {

                @Override
                public void run() {
                    System.out.println("开始线程阻塞");
                    LockSupport.park();
                    System.out.println("结束线程阻塞");
                }
            }
        }
        ```

        结果

        ```
        开始线程唤醒
        开始线程阻塞
        结束线程唤醒
        结束线程阻塞
        ```

        LockSupport.park();可以用来阻塞当前线程,park是停车的意思，把运行的线程比作行驶的车辆，线程阻塞则相当于汽车停车，相当直观。该方法还有个变体LockSupport.park(Object blocker),指定线程阻塞的对象blocker，该对象主要用来排查问题。方法LockSupport.unpark(Thread thread)用来唤醒线程，因为需要线程作参数，所以可以指定线程进行唤醒。

2. LockSupport的其他特色

    1. 可以先唤醒线程再阻塞线程

        在阻塞线程前睡眠1秒中，使唤醒动作先于阻塞发生，看看会发生什么

        ```java
        public class LockSupportTest {

            public static void main(String[] args) {
                Thread parkThread = new Thread(new ParkThread());
                parkThread.start();
                System.out.println("开始线程唤醒");
                LockSupport.unpark(parkThread);
                System.out.println("结束线程唤醒");

            }

            static class ParkThread implements Runnable{

                @Override
                public void run() {
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("开始线程阻塞");
                    LockSupport.park();
                    System.out.println("结束线程阻塞");
                }
            }
        }
        ```

        结果

        ```
        开始线程唤醒
        结束线程唤醒
        开始线程阻塞
        结束线程阻塞
        ```

        先唤醒指定线程,然后阻塞该线程，但是线程并没有真正被阻塞而是正常执行完后退出了。这是怎么回事？我们试着在改动下代码,先唤醒线程两次，在阻塞线程两次，看看会发生什么。

    2. 先唤醒线程两次再阻塞两次会发生什么

        ```java
        public class LockSupportTest {

            public static void main(String[] args) {
                Thread parkThread = new Thread(new ParkThread());
                parkThread.start();
                for(int i=0;i<2;i++){
                    System.out.println("开始线程唤醒");
                    LockSupport.unpark(parkThread);
                    System.out.println("结束线程唤醒");
                }
            }

            static class ParkThread implements Runnable{

                @Override
                public void run() {
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    for(int i=0;i<2;i++){
                        System.out.println("开始线程阻塞");
                        LockSupport.park();
                        System.out.println("结束线程阻塞");
                    }
                }
            }
        }

        可以看到线程被阻塞导致程序一直无法结束掉。对比上面的例子，我们可以得出一个匪夷所思的结论，先唤醒线程，在阻塞线程，线程不会真的阻塞；但是先唤醒线程两次再阻塞两次时就会导致线程真的阻塞。那么这到底是为什么？

3. LockSupport阻塞和唤醒线程原理浅析

    既然是浅析，那就不抠底层细节，只讲关键，细节可能有疏漏和不到位的地方。
    每个线程都有Parker实例,如下面的代码所示

    ```c
    class Parker : public os::PlatformParker {
    private:
    volatile int _counter ;
    ...
    public:
    void park(bool isAbsolute, jlong time);
    void unpark();
    ...
    }
    class PlatformParker : public CHeapObj<mtInternal> {
    protected:
        pthread_mutex_t _mutex [1] ;
        pthread_cond_t  _cond  [1] ;
        ...
    }
    ```

    LockSupport就是通过控制变量_counter来对线程阻塞唤醒进行控制的。原理有点类似于信号量机制。

    * 当调用park()方法时，会将_counter置为0，同时判断前值，小于1说明前面被unpark过,则直接退出，否则将使该线程阻塞。
    * 当调用unpark()方法时，会将_counter置为1，同时判断前值，小于1会进行线程唤醒，否则直接退出。

    形象的理解，线程阻塞需要消耗凭证(permit)，这个凭证最多只有1个。
    当调用park方法时，如果有凭证，则会直接消耗掉这个凭证然后正常退出；但是如果没有凭证，就必须阻塞等待凭证可用；
    而unpark则相反，它会增加一个凭证，但凭证最多只能有1个。

    为什么可以先唤醒线程后阻塞线程？
    因为unpark获得了一个凭证,之后调用park因为有凭证消费，故不会阻塞。
    为什么唤醒两次后阻塞两次会阻塞线程。
    因为凭证的数量最多为1，连续调用两次unpark和调用一次unpark效果一样，只会增加一个凭证；而调用两次park却需要消费两个凭证。

4. 总结

    LockSupport是JDK中用来实现线程阻塞和唤醒的工具。使用它可以在任何场合使线程阻塞，可以指定任何线程进行唤醒，并且不用担心阻塞和唤醒操作的顺序，但要注意连续多次唤醒的效果和一次唤醒是一样的。
    JDK并发包下的锁和其他同步工具的底层实现中大量使用了LockSupport进行线程的阻塞和唤醒，掌握它的用法和原理可以让我们更好的理解锁和其它同步工具的底层实现。


原文：https://www.cnblogs.com/takumicx/p/9328459.html