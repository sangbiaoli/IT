# Centos7下关于memcached的安装和简单使用
## 前言：memcached的介绍
> Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的hashmap。其守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。

> 对于强化版的memcached：Redis，在我的前两篇博客有介绍到它的安装和使用：Redis在Centos7上的安装部署 、Centos7下安装php-redis扩展及简单使用，如果大家想了解memcached和redis的异同和在什么情况下我们应该怎么选择缓存系统，以下这篇博客相信对大家会有一个很好的启发：缓存技术PK：选择Memcached还是Redis？，感谢这篇博客的作者的精辟分析。

在本篇博客中，我会带领大家在Centos7下安装和使用memcached

## 服务端的安装  
1. 安装    
在这里，由于用编译安装memcached服务端过于复杂，因此我选用依赖管理工具yum来实现 memcached 的服务端安装：
```shell
[root@localhost /]# yum install -y memcached
```
* -y 表示自动应答，即默认安装所有需要用到的依赖包,在这一步之后，我们就安装完了。
2. 启动  
我们尝试去启动一下memcached：
```bash
[root@localhost /]# /usr/bin/memcached -d -l 127.0.0.1 -p 11211 -m 150 -u root  /tmp/memcached.pid
```
* -d 守护进程模式（退出终端窗口之后使程序还在运行），
* -l 指定IP地址127.0.0.1 ，
* -p 指定端口号11211，
* -m 为memcached分配多少内存（单位：M），
* -u 指定使用哪个用户启动memcached

&emsp;查看memcached是否在运行：
```bash
[root@localhost /]# ps -ef | grep memcached
//或
[root@localhost /]# pstree -p | grep memcached
```
&emsp;如果能够看到存在memcached进程，那就说明我们的 memcached 服务端已经安装成功了。

3. 添加启动脚本
```bash
# http://blog.phpha.com  
# 以下内容摘自互联网  
vi /etc/rc.d/init.d/memcached  
#!/bin/sh  
#  
# memcached:    MemCached Daemon  
# chkconfig:    - 90 25  
# description:  MemCached Daemon  
# Source function library.  
. /etc/rc.d/init.d/functions  
. /etc/sysconfig/network  
#[ ${NETWORKING} = "no" ] && exit 0  
#[ -r /etc/sysconfig/dund ] || exit 0  
#. /etc/sysconfig/dund  
#[ -z "$DUNDARGS" ] && exit 0  
start()  
{  
        echo -n $"Starting memcached: "  
        daemon $MEMCACHED -u daemon -d -m 64 -l 127.0.0.100 -p 11211 -c 128 -P /tmp/memcached.pid  
        echo  
}  
stop()  
{  
        echo -n $"Shutting down memcached: "  
        killproc memcached  
        echo  
}  
MEMCACHED="/usr/local/memcached/bin/memcached"  
[ -f $MEMCACHED ] || exit 1  
# See how we were called.  
case "$1" in  
        start)  
                start  
                ;;  
        stop)  
                stop  
                ;;  
        restart)  
                stop  
                sleep 3  
                start  
                ;;  
        *)  
                echo $"Usage: $0 {start|stop|restart}"  
                exit 1  
esac  
exit 0
```

## 客户端的安装

&emsp;&emsp;**libmemcached-1.0.18.tar.gz**
```bash
[root@localhost /]# cd /usr/local/src   #我的所有源码包习惯放在该目录下
[root@localhost src]# wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz  #下载libmemcached源码包
[root@localhost src]# tar -zxvf libmemcached-1.0.18.tar.gz
[root@localhost src]# cd libmemcached-1.0.18/
[root@localhost libmemcached-1.0.18]# ./configure --prefix=/usr/local/libmemcached
[root@localhost libmemcached-1.0.18]# make && make install  #如果报错，可能要更新gcc
[root@localhost libmemcached-1.0.18]#yum -y update gcc  
[root@localhost libmemcached-1.0.18]#yum -y install gcc+ gcc-c++  
[root@localhost libmemcached-1.0.18]#make clean all LDFLAGS="-L/usr/lib64 -L/lib64"
```
&emsp;&emsp;**memcached-2.2.0.tgz**
```bash
[root@localhost libmemcached-1.0.18]# cd ..
[root@localhost src]# wget http://pecl.php.net/get/memcached-2.2.0.tgz  #下载memcached源码包（wget下载不了，就手动下载并上传）
[root@localhost src]# tar -zxvf memcached-2.2.0.tgz
[root@localhost src]# cd  memcached-2.2.0
[root@localhost memcached-2.2.0]# /usr/local/php/bin/phpize (或 /usr/bin/phpize) 
[root@localhost memcached-2.2.0]# ./configure -with-php-config=/usr/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached --disable-memcached-sasl
[root@localhost memcached-2.2.0]# make && make install
```
* 使用安装php时生成的 phpize 来生成 configure 配置文件
* -with-php-config 指定 php-config，该文件与 phpize 所在目录相同， 
* –with-libmemcached-dir 指定 libmemcached 安装目录，就刚才我们 –prefix 那个目录 ，
* –disable-memcached-sasl 说明我们系统不支持sasl.h
* 如果安装成功，会提示：Installing shared extension:/usr/local/php/lib/extensions/no-debug-non-zts-20160524/ 等类信息
* <font color=red>error: memcached support requires ZLIB</font> 需要安装
* <font color=green>yum install zlib zlib-devel</font>

## php-devel安装
```bash
[root@localhost src]# yum install php-devel
```
为PHP安装 php-memcached 扩展   
接下来，我们编辑php配置文件php.ini，你可以用 whereis php.ini 查看所在位置（我的在 /etc/php.ini ），把 php-memcached 扩展加到配置文件。
在 php.ini 中添加以下内容：
```bash
extension=memcached.so
```
重启apache服务器，使配置生效
```bash
[root@localhost memcached-2.2.0]# systemctl restart httpd.service
```
重启完之后，检查是否安装完成php-memcached扩展
```bash
[root@localhost memcached-2.2.0]# echo "<?php echo phpinfo() ?>">>/home/www/index.php(这里web目录如果没改的话是在 /var/www/html/)
```
在浏览器地址栏输入 127.0.0.1，查看php扩展，如果有以下图片所示，则表示安装成功

