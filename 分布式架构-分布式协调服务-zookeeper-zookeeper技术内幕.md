## zookeeper技术内幕

1. 分布式锁解决方案

2. Watcher核心机制

3. Zab协议

    Zookeeper使用了一种Zookeeper Atomic Broadcast(ZAB，Zookeeper原子消息广播协议)的协议作为其数据一致性的核心算法。

    1. Zab协议的核心

        所有事物请求必须有一个全局唯一的服务器来协调处理，这样的服务器被称为Leader服务器，而雨下的其他服务器则成为Follower服务器。Leader服务器负责将一个客户端事物请求转换成一个事物Proposal(提议)，并将该Proposal分发给集群中所有的Follower服务器。之后Leader服务器需要等待所有Follower服务器的反馈，一旦超过半数的Follower服务器进行正确的反馈后，那么Leader机会再次向所有的Follower服务器分发Commit消息，要求器将前一个Proposal进行提交。

    2. 消息广播

        ZAB协议的消息广播过程使用的是一个原子广播协议，类似于一个二阶段提交过程。对客户端的事务请求，Leader服务器为其生成对应的事务Proposal，并将其发送给群中其余所有的机器，然后再分别收集各自的选票，最后进行事务提交，如图

        ![](zookeeper/zookeeper-tech-broadcast.jpg)

        在整个消息广播过程中，Leader服务器会为每个事务请求生成对应的Proposal来进行广播，并且在广播事务Proposal之前，Leader服务器会首先为这个事务Proposal分配一个全局单调递增的唯一ID，我们称之为事务ID(即ZXID)。由于ZAB协议需要保证每一个消息严格的因果关系，因此必须将每一个事务Proposal按照其ZXID的先后顺序来进行排序与处理。

    3. 崩溃恢复过程

        在正常情况下运行非常良好，但是一旦Leader服务器出现崩溃，或者说由于网络原因导致Leader服务器失去了与过半Follower的联系，那么就会进入崩溃恢复模式。

        1. 选举出来的Leader服务器拥有集群中所有机器最高编号(即ZXID最大)的事务Proposal，那么可以保证这个新选举出来的Leader一定具有所有已经提交的提案。

        2. 完成Leader选举之后，在正式开始工作(即接收客户端的事务请求，然后提出新的提案)之前，Leader服务器会首先确认事务日志中的所有Proposal是否都已经被群集中过半的机器提交了，即是否完成数据同步。 *Leader服务器会为每一个Follower服务器都准备一个队列，并将那些没有被各Follower服务器同步的事务以Proposal消息的形式逐个发送给Follower服务器，并且在每一个Proposal消息后面紧接着在发送一个Commit消息，以表示该事务已经被提交*

        3. ZXID设计，ZXID是一个64位的数字，新增事务Proposal时，低32位加1。高32为代表Leader周期epoch的编号，每产生新Leader服务器，低32置0，Leader服务器从本地日志中取出最大事务Proposal的ZXID，解析出对应的epoch值，然后加1，最为最新的epoch。这个策略能有效地避免不同的Leader服务器错误地使用相同的ZXID编号提出不一样的事务Proposal的异常情况。


原文：从Paxos到Zookeeper++分布式一致性原理与实践.pdf