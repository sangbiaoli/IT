Java多线程并发工具类

目录

AbstractQueuedSynchronizer

一、闭锁CountDownLatch

二、栅栏CyclicBarrier

三、信号量Semaphore

四、FutureTask

五、Exchanger

 
#### AbstractQueuedSynchronizer

是一个用于构建锁和同步器的框架；CountDownLatch、Semaphore、FutureTask都是基于AQS构建的。

#### 一、闭锁CountDownLatch

1. 简介

    多个线程执行完后再执行下一步操作

2. 举例

    ```java
    public class CountDownLatchUtils {
        private static final int count = 5;

        public static void main(String[] args) {

            final CountDownLatch countDownLatch = new CountDownLatch(count);
            final ExecutorService executorService = Executors.newFixedThreadPool(count);

            for (int i = 0; i < count; i++) {
                final int finalI = i;
                Future<?> future = executorService.submit(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            dosomething(finalI);
                            countDownLatch.countDown();// 执行一次减小1
                            System.out.println(Thread.currentThread().getName() + " over");
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                });
            }

            try {
                countDownLatch.await();// 阻塞
                System.out.println("所有线程执行成功！！！dosomething...");

                executorService.shutdown();
                if (!executorService.awaitTermination(10 * 1000, TimeUnit.MILLISECONDS)) {
                    executorService.shutdownNow();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

        public static void dosomething(int i) throws InterruptedException {
            Thread.sleep((i + 1) * 2000);
            System.out.println(Thread.currentThread().getName() + "任务执行完成");
        }
    }
    ```

    输出结果
    ```
    pool-1-thread-1任务执行完成
    pool-1-thread-1 over
    pool-1-thread-2任务执行完成
    pool-1-thread-2 over
    pool-1-thread-3任务执行完成
    pool-1-thread-3 over
    pool-1-thread-4任务执行完成
    pool-1-thread-4 over
    pool-1-thread-5任务执行完成
    pool-1-thread-5 over
    所有线程执行成功！！！dosomething...
    ```
#### 二、栅栏CyclicBarrier

1. 简介

    1. CyclicBarrier用于等待线程。可循环使用（Cyclic）的屏障（Barrier）。让一组线程到达一个屏障（同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程继续干活。

    2. 与CountDownLatch不同的是该barrier在释放等待线程后可以重用

    3. 支持一个可选的Runnable命令，在一组线程中的最后一个线程到达之后（但在释放所有线程之前），该命令只在每个屏障点运行一次

2. 举例

    ```java
    public class CyclicBarrierUtils {

        public static void main(String[] args) throws InterruptedException {

            ExecutorService consumerExe = Executors.newFixedThreadPool(2);
            final CyclicBarrier cyclicBarrier = new CyclicBarrier(2, new Runnable() {
                @Override
                public void run() {
                    System.out.println("栅栏打开！继续执行任务！");
                }
            });

            for (int i = 0; i < 4; i++) {
                final int finalI = i;
                consumerExe.submit(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            dosomething(finalI);
                            int i = cyclicBarrier.await();
                            System.out.println(Thread.currentThread().getName() + "等待完成i;返回码：" + i);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        } catch (BrokenBarrierException e) {
                            e.printStackTrace();
                        }
                    }
                });
            }
            consumerExe.shutdown();
            boolean flag = consumerExe.awaitTermination(10 * 1000, TimeUnit.MILLISECONDS);
            if (!flag) {
                consumerExe.shutdownNow();
            }
        }

        public static void dosomething(int i) throws InterruptedException {
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + "任务执行完成");
        }
    }
    ```

    输出结果
    ```
    pool-1-thread-2任务执行完成
    pool-1-thread-1任务执行完成
    栅栏打开！继续执行任务！
    pool-1-thread-1等待完成i;返回码：0
    pool-1-thread-2等待完成i;返回码：1
    pool-1-thread-2任务执行完成
    pool-1-thread-1任务执行完成
    栅栏打开！继续执行任务！
    pool-1-thread-1等待完成i;返回码：0
    pool-1-thread-2等待完成i;返回码：1
    ```
####　三、信号量Semaphore

1. 简介

    用于资源的多副本的并发访问控制,线程访问共享资源，先得到信号量。

    若大于1表示共享资源可访问，信号量减去1在访问共享资源。如果计数器值为0,线程进入休眠。当某个线程使用完共享资源后，释放信号量，并将信号量内部的计数器加1，之前进入休眠的线程将被唤醒并再次试图获得信号量。

    内部维护了一个计数器，其值为可以访问的共享资源的个数。

2. 举例
    
    4个碗筷，6个人吃饭

    ```java
    public class SemaphoreUtils {

        private final static Semaphore semaphore = new Semaphore(4);
        private final static ExecutorService produceExe = Executors.newCachedThreadPool();

        public static void main(String[] args) throws InterruptedException {

            for (int i = 0; i < 6; i++) {
                produceExe.submit(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            eat();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }
                });

            }
            produceExe.shutdown();
            produceExe.awaitTermination(10, TimeUnit.SECONDS);
        }

        public static void eat() throws InterruptedException {
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName() + ":得到碗筷，开始吃饭");
            Thread.sleep(2000);
            semaphore.release();
            System.out.println(Thread.currentThread().getName() + ":释放碗筷。。。");
        }
    }
    ```

    输出结果
    ```
    pool-1-thread-2:得到碗筷，开始吃饭
    pool-1-thread-1:得到碗筷，开始吃饭
    pool-1-thread-3:得到碗筷，开始吃饭
    pool-1-thread-4:得到碗筷，开始吃饭
    pool-1-thread-1:释放碗筷。。。
    pool-1-thread-6:得到碗筷，开始吃饭
    pool-1-thread-2:释放碗筷。。。
    pool-1-thread-3:释放碗筷。。。
    pool-1-thread-5:得到碗筷，开始吃饭
    pool-1-thread-4:释放碗筷。。。
    pool-1-thread-5:释放碗筷。。。
    pool-1-thread-6:释放碗筷。。。
    ```

#### 四、FutureTask

1. 简介
    
    用于异步执行某个只执行一次的任务，可以确保即使调用了多次run方法，它都只会执行一次Runnable或者Callable任务

2. 举例

    ```java
    public class FutrueTaskUtils {
        public static void main(String[] args) throws InterruptedException {

            FutureTask<String> futureTask = new FutureTask<String>(new Callable<String>() {
                @Override
                public String call() throws Exception {
                    Thread.sleep(5 * 1000);
                    System.out.println("futureTask任务执行成功");
                    return "SUCCESS";
                }
            });
            ExecutorService executorService = Executors.newFixedThreadPool(2);
            for (int i = 0; i < 4; i++) {
                final int finalI = i;
                executorService.submit(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            dosomething(finalI);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                });
                executorService.submit(futureTask);
            }
            try {
                System.out.println("futureTask任务，阻塞等待。。。");
                String str = futureTask.get();
                if ("SUCCESS".equals(str)) {
                    System.out.println("futureTask任务完成");
                    executorService.shutdown();
                }
                System.out.println(str);
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }

        public static void dosomething(int i) throws InterruptedException {
            Thread.sleep((i + 1) * 2000);
            System.out.println(Thread.currentThread().getName() + "任务执行完成");
        }
    }
    ```

    输出结果
    ```
    futureTask任务，阻塞等待。。。
    pool-1-thread-1任务执行完成
    futureTask任务执行成功
    futureTask任务完成
    SUCCESS
    pool-1-thread-1任务执行完成
    pool-1-thread-2任务执行完成
    pool-1-thread-1任务执行完成
    ```
#### 五、Exchanger

1. 简介
    
    Exchanger可以在两个线程之间交换数据，只能是2个线程，他不支持更多的线程之间互换数据。

    1. 线程A调用Exchange对象的exchange()方法后，陷入阻塞状态。

    2. 直到线程B也调用了exchange()方法，然后以线程安全的方式交换数据

    3. 线程A和B继续运行

2. 举例

    两个线程分别生产一根筷子，最后组合成一双

    ```java
    public class ExChangerUtils {
        public static void main(String[] args) {

            final int num = 10;

            final Exchanger<Chopsticks> exchanger = new Exchanger();

            final ConcurrentLinkedQueue<Chopsticks> dataQueue = new ConcurrentLinkedQueue();
            final ConcurrentLinkedQueue<Chopsticks> resultQueue = new ConcurrentLinkedQueue();

            for (int i = 0; i < num; i++) {
                Chopsticks chopsticks = new Chopsticks();
                dataQueue.offer(chopsticks);
            }

            ExecutorService producerExe1 = Executors.newSingleThreadExecutor();
            ExecutorService producerExe2 = Executors.newSingleThreadExecutor();

            producerExe1.submit(new Runnable() {
                @Override
                public void run() {
                    while (true) {

                        if (!dataQueue.isEmpty()) {
                            Chopsticks chopsticks = dataQueue.poll();
                            chopsticks.setChopstick1("chopstick1");
                            try {
                                Chopsticks changeChop = exchanger.exchange(chopsticks);
                                changeChop.setChopstick1("chopstick1");
                                if (!StringUtils.isBlank(changeChop.getChopstick2())) {
                                    resultQueue.offer(changeChop);
                                }
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        } else {
                            break;
                        }

                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }
                    System.out.println("thread1 over");

                }
            });

            producerExe2.submit(new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        if (!dataQueue.isEmpty()) {
                            Chopsticks chopsticks = dataQueue.poll();
                            chopsticks.setChopstick2("chopstick2");
                            try {
                                Chopsticks changeChop = exchanger.exchange(chopsticks);
                                changeChop.setChopstick2("chopstick2");
                                if (!StringUtils.isBlank(changeChop.getChopstick1())) {
                                    resultQueue.offer(changeChop);
                                }
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        } else {
                            break;
                        }
                    }
                    System.out.println("thread2 over");
                }
            });

            new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        if (resultQueue.size() == num) {
                            for (Chopsticks chopsticks : resultQueue) {
                                System.out.println(chopsticks.toString());
                            }
                            break;
                        }
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                }
            }.run();

        }

        public static class Chopsticks {
            public String chopstick1;
            public String chopstick2;

            public String getChopstick1() {
                return chopstick1;
            }

            public void setChopstick1(String chopstick1) {
                this.chopstick1 = chopstick1;
            }

            public String getChopstick2() {
                return chopstick2;
            }

            public void setChopstick2(String chopstick2) {
                this.chopstick2 = chopstick2;
            }

            @Override
            public String toString() {
                final StringBuffer sb = new StringBuffer("Chopsticks{");
                sb.append("chopstick1='").append(chopstick1).append('\'');
                sb.append(", chopstick2='").append(chopstick2).append('\'');
                sb.append('}');
                return sb.toString();
            }
        }
    }
    ```

    输出结果
    ```
    thread2 over
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    Chopsticks{chopstick1='chopstick1', chopstick2='chopstick2'}
    thread1 over
    ```

原文：https://blog.csdn.net/CoderLai/article/details/83153100