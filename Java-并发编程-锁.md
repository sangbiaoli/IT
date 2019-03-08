## Lock、Condition、ReentrantLock、ReentrantReadWriteLock

#### Lock

1. 背景

    锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源。在Java SE5之前，Java程序是靠synchronized关键字实现锁功能的，使用synchronized关键字将会隐式地获取锁，但是它将锁的获取和释放固化了，也就是先获取再释放，这种方式简化了同步的管理，可是扩展性没有显式地锁获取和释放来的好。例如，考虑下面这样一个情景：

    针对一个场景，使用synchronized进行锁获取和释放，先获得锁A，然后获得锁B，当锁B获取之后，释放锁A同时获取锁C，当锁C获得后，在释放B同时获取锁D，以此类推。这时，使用synchronized关键字就不那么容易实现了，而使用显式地锁获取和释放则很简单。

    在Java SE5之后，并发包中新增了Lock接口以及相关实现类，用来实现锁功能，它提供了与synchronized类似的同步功能，只是在使用时需要显式地获取和释放锁。虽然它缺少了隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等多种synchronized所不具备的特性。

2. Lock接口

    Lock接口的使用很简单，常见的使用方式如下代码所示：
    ```java
    Lock lock = new ReentrantLock();
    lock.lock();
    
    try {
        // do something...
    } finally {
        lock.unlock();
    }
    ```

    在使用Lock锁的时候要**注意**以下几点：

    * 要在finally块中释放锁，目的是保证在获取到锁之后，最终能够被释放。
    * 不要将获取锁的过程写在try块中，因为如果在获取锁（可以是自定义的锁）时发生了异常，在异常抛出的同时，也会导致锁的无故释放。

    Lock接口提供的synchronized不具备的**主要特征**如下：
    特  性|描  述
    --|--
    尝试非阻塞地获取锁|当前线程尝试获取锁，如果这一刻锁没有被其他线程获取到，则成功获取并持有锁。
    能被中断地获取锁|与synchronized不同，获取到锁的线程能够相应中断，当获取到锁的线程被中断时，中断异常将会被抛出，同时锁会被释放。
    超时获取锁|在指定的截止时间之前获取锁，如果截止时间到了仍没有获取到锁，则返回。

    Lock是一个接口，它定义了锁获取和释放的基本操作，**Lock的API**如下所示：
    方法名称|描  述
    --|--
    void lock()|获取锁，调用该方法当前线程会获取锁，当锁获取后，从该方法返回。
    void lockInterruptibly() throws InterruptedException|可中断地获取锁，与lock()方法的不同之处在于该方法会响应中断，即在锁的获取中可以中断当前线程。
    boolean tryLock()|尝试非阻塞地获取锁，调用该方法后立即返回，如果能够获取锁则返回true，否则返回false。
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException|超时地获取锁，当前线程在以下3种情况下会返回：</br>1. 当前线程在超时时间内获取了锁 </br>2. 当前线程在超时时间内被中断 </br>3. 超时时间结束，返回false
    void unlock()|释放锁
    Condition newCondition()|获取等待通知组件，在组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的await()方法，而调用后，当前线程将会释放锁。

    Lock接口的实现基本都是通过聚合了一个AbstractQueuedSynchronizer同步器的子类来完成线程访问控制的，下面介绍Lock的实现类。





原文：https://blog.csdn.net/qq_38293564/article/details/80476659