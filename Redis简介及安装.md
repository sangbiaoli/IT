### Redis简介
Redis是一个开放源代码（BSD许可证），在内存数据结构存储。用作数据库、缓存和消息代理。它支持数据结构，如字符串、散列、列表、集合、带有范围查询的排序集、位图、超对数和地理空间索引以及radius查询。

Redis是当前比较热门的NOSQL系统之一，它是一个key-value存储系统。和Memcache类似，但很大程度补偿了Memcache的不足，它支持存储的value类型相对更多，包括string、list、set、zset和hash。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作。在此基础上，Redis支持各种不同方式的排序。

和Memcache一样，Redis数据都是缓存在计算机内存中，不同的是，Memcache只能将数据缓存到内存中，无法自动定期写入硬盘，这就表示，一断电或重启，内存清空，数据丢失。所以Memcache的应用场景适用于缓存无需持久化的数据。而Redis不同的是它会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，实现数据的持久化。

## Redis安装
1、下载redis安装包
```bash
cd /usr/local
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
```
 
2、解压缩redis并安装
```bash
tar -zxvf redis-4.0.9.tar.gz
cd redis-4.0.9
make
```

<font color="yellow">如果运行出现 make：gcc：Commond not found错误</font>

一般出现这种错误是因为安装系统的时候使用的是最小化mini安装，系统没有安装make、vim等常用命令，直接yum安装下即可。
```
yum -y install gcc automake autoconf libtool make
```

3、 启动服务端
```bash
cd /usr/local/redis-4.0.9/src
./redis-server & #启动
```

4、客户端使用
新打开一个tab页
```bash
cd /usr/local/redis-4.0.9/src
 ./redis-cli #客户端连接
set test 123    #设置值
get test    #获取值
```