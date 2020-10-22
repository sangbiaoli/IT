## Redis分布式锁实现秒杀业务(乐观锁、悲观锁)

一、业务场景

    所谓秒杀，从业务角度看，是短时间内多个用户“争抢”资源，这里的资源在大部分秒杀场景里是商品；将业务抽象，技术角度看，秒杀就是多个线程对资源进行操作，所以实现秒杀，就必须控制线程对资源的争抢，既要保证高效并发，也要保证操作的正确。

二、一些可能的实现

    刚才提到过，实现秒杀的关键点是控制线程对资源的争抢，根据基本的线程知识，可以不加思索的想到下面的一些方法：

    1. 秒杀在技术层面的抽象应该就是一个方法，在这个方法里可能的操作是将商品库存-1，将商品加入用户的购物车等等，在不考虑缓存的情况下应该是要操作数据库的。那么最简单直接的实现就是在这个方法上加上synchronized关键字，通俗的讲就是锁住整个方法；

    2. 锁住整个方法这个策略简单方便，但是似乎有点粗暴。可以稍微优化一下，只锁住秒杀的代码块，比如写数据库的部分；

    3. 既然有并发问题，那我就让他“不并发”，将所有的线程用一个队列管理起来，使之变成串行操作，自然不会有并发问题。

    上面所述的方法都是有效的，但是都不好。为什么？第一和第二种方法本质上是“加锁”，但是锁粒度依然比较高。什么意思？试想一下，如果两个线程同时执行秒杀方法，这两个线程操作的是不同的商品,从业务上讲应该是可以同时进行的，但是如果采用第一二种方法，这两个线程也会去争抢同一个锁，这其实是不必要的。第三种方法也没有解决上面说的问题。

    那么如何将锁控制在更细的粒度上呢？可以考虑为每个商品设置一个互斥锁，以和商品ID相关的字符串为唯一标识，这样就可以做到只有争抢同一件商品的线程互斥，不会导致所有的线程互斥。分布式锁恰好可以帮助我们解决这个问题。

三、具体的实现

    需要考虑的问题

    1. 用什么操作redis？幸亏redis已经提供了jedis客户端用于java应用程序，直接调用jedis API即可。

    2. 怎么实现加锁？“锁”其实是一个抽象的概念，将这个抽象概念变为具体的东西，就是一个存储在redis里的key-value对，key是于商品ID相关的字符串来唯一标识，value其实并不重要，因为只要这个唯一的key-value存在，就表示这个商品已经上锁。

    3. 如何释放锁？既然key-value对存在就表示上锁，那么释放锁就自然是在redis里删除key-value对。

    4. 阻塞还是非阻塞？笔者采用了阻塞式的实现，若线程发现已经上锁，会在特定时间内轮询锁。

    5. 如何处理异常情况？比如一个线程把一个商品上了锁，但是由于各种原因，没有完成操作（在上面的业务场景里就是没有将库存-1写入数据库），自然没有释放锁，这个情况笔者加入了锁超时机制，利用redis的expire命令为key设置超时时长，过了超时时间redis就会将这个key自动删除，即强制释放锁（可以认为超时释放锁是一个异步操作，由redis完成，应用程序只需要根据系统特点设置超时时间即可）。

四、乐观锁实现秒杀系统

    我们知道大多数是基于数据版本（version）的记录机制实现的。即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个”version”字段来实现读取出数据时，将此版本号一同读出，之后更新时，对此版本号加1。此时，将提交数据的版本号与数据库表对应记录的当前版本号进行比对，如果提交的数据版本号大于数据库当前版本号，则予以更新，否则认为是过期数据。redis中可以使用watch命令会监视给定的key，当exec时候如果监视的key从调用watch后发生过变化，则整个事务会失败。也可以调用watch多次监视多个key。这样就可以对指定的key加乐观锁了。注意watch的key是对整个连接有效的，事务也一样。如果连接断开，监视和事务都会被自动清除。当然了exec，discard，unwatch命令都会清除连接中的所有监视。

    Redis中的事务(transaction)是一组命令的集合。事务同命令一样都是Redis最小的执行单位，一个事务中的命令要么都执行，要么都不执行。Redis事务的实现需要用到 MULTI 和 EXEC 两个命令，事务开始的时候先向Redis服务器发送 MULTI 命令，然后依次发送需要在本次事务中处理的命令，最后再发送 EXEC 命令表示事务命令结束。Redis的事务是下面4个命令来实现 ：

    1. multi，开启Redis的事务，置客户端为事务态。

    2. exec，提交事务，执行从multi到此命令前的命令队列，置客户端为非事务态。

    3. discard，取消事务，置客户端为非事务态。

    4. watch,监视键值对，作用时如果事务提交exec时发现监视的监视对发生变化，事务将被取消。

五、代码实现

    ```java
    import java.util.List;
    import java.util.Set;
    import java.util.concurrent.Executor;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    import javax.transaction.Transaction;
    import redis.clients.jedis.Jedis;

    /**
    * 乐观锁实现秒杀系统
    *
    */
    public class OptimisticLockTest {

        public static void main(String[] args) {
            long startTime = System.currentTimeMillis();
            initProduct();
            initClient();
            printResult();
            long endTime = System.currentTimeMillis();
            long time = endTime - startTime;
            System.out.println("程序运行时间 ： " + (int)time + "ms");
        }

        /**
        * 初始化商品
        * @date 2017-10-17
        */
        public static void initProduct() {
            int prdNum = 100;//商品个数
            String key = "prdNum";
            String clientList = "clientList";//抢购到商品的顾客列表
            Jedis jedis = RedisUtil.getInstance().getJedis();
            if (jedis.exists(key)) {
                jedis.del(key);
            }
            if (jedis.exists(clientList)) {
                jedis.del(clientList);
            }
            jedis.set(key, String.valueOf(prdNum));//初始化商品
            RedisUtil.returnResource(jedis);
        }

        /**
        * 顾客抢购商品（秒杀操作）
        * @date 2017-10-17
        */
        public static void initClient() {
            ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
            int clientNum = 10000;//模拟顾客数目
            for (int i = 0; i < clientNum; i++) {
                cachedThreadPool.execute(new ClientThread(i));//启动与顾客数目相等的消费者线程
            }
            cachedThreadPool.shutdown();//关闭线程池
            while (true) {
                if (cachedThreadPool.isTerminated()) {
                    System.out.println("所有的消费者线程均结束了");    
                    break;   
                }
                try {
                    Thread.sleep(100);
                } catch (Exception e) {
                    // TODO: handle exception
                }
            }
        }

        /**
        * 打印抢购结果
        * @date 2017-10-17
        */
        public static void printResult() {
            Jedis jedis = RedisUtil.getInstance().getJedis();
            Set<String> set = jedis.smembers("clientList");
            int i = 1;
            for (String value : set) {
                System.out.println("第" + i++ + "个抢到商品，"+value + " "); 
            }
            RedisUtil.returnResource(jedis);
        }


        /**
        * 内部类：模拟消费者线程
        */
        static class ClientThread implements Runnable{

            Jedis jedis = null;
            String key = "prdNum";//商品主键
            String clientList = "clientList";//抢购到商品的顾客列表主键
            String clientName;

            public ClientThread(int num){
                clientName = "编号=" + num;
            }

    //      1.multi，开启Redis的事务，置客户端为事务态。 
    //      2.exec，提交事务，执行从multi到此命令前的命令队列，置客户端为非事务态。 
    //      3.discard，取消事务，置客户端为非事务态。 
    //      4.watch,监视键值对，作用是如果事务提交exec时发现监视的键值对发生变化，事务将被取消。 
            @Override
            public void run() {
                try {
                    Thread.sleep((int)Math.random()*5000);//随机睡眠一下
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                while(true){
                    System.out.println("顾客：" + clientName + "开始抢购商品");
                    jedis = RedisUtil.getInstance().getJedis();
                    try {
                        jedis.watch(key);//监视商品键值对，作用时如果事务提交exec时发现监视的键值对发生变化，事务将被取消
                        int prdNum = Integer.parseInt(jedis.get(key));//当前商品个数
                        if (prdNum > 0) {
                            Transaction transaction = (Transaction) jedis.multi();//开启redis事务
                            ((Jedis) transaction).set(key,String.valueOf(prdNum - 1));//商品数量减一
                            List<Object> result = ((redis.clients.jedis.Transaction) transaction).exec();//提交事务(乐观锁：提交事务的时候才会去检查key有没有被修改)
                            if (result == null || result.isEmpty()) {
                                System.out.println("很抱歉，顾客:" + clientName + "没有抢到商品");// 可能是watch-key被外部修改，或者是数据操作被驳回
                            }else {
                                jedis.sadd(clientList, clientName);//抢到商品的话记录一下
                                System.out.println("恭喜，顾客:" + clientName + "抢到商品");  
                                break; 
                            }
                        }else {
                            System.out.println("很抱歉，库存为0，顾客:" + clientName + "没有抢到商品");  
                            break; 
                        }
                    } catch (Exception e) {
                        // TODO: handle exception
                    }finally{
                        jedis.unwatch();
                        RedisUtil.returnResource(jedis);
                    }
                }
            }
        }
    }
    ```

原文：https://www.cnblogs.com/jasonZh/p/9522772.html