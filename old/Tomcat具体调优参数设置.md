## tomcat性能调优

tomcat通常是作为开发环境的容器，其配置也默认是开发环境的，在性能提升方面还有很大空间。

本文主要从三个方面介绍一下tomcat性能调优：内存，线程数，IO。

#### 一、Tomcat启动行参数的优化

32位操作系统与64位操作系统中JVM的对比
操作系统|操作系统位数|内存限制|解决办法
--|--|--|--
Winxp|32|4GB|超级兔子
Win7|32|4GB|可以通过设置/PAE
Win2003|32|可以突破4GB达16GB|必需要装win2003 advanced server且要打上sp2补丁
Win7|64|无限制|机器能插多少内存，系统内存就能支持到多大
Win2003|64|无限制|机器能插多少内存，系统内存就能支持到多大
Linux|64|无限制|机器能插多少内存，系统内存就能支持到多大
Unix|64|无限制|机器能插多少内存，系统内存就能支持到多大

主要是设定虚拟机的server启动方式，以及堆内存的初始分配大小，垃圾收集机制，线程最大堆栈配置数，新生代内存大小等等

* -server

    tomcat默认是以一种叫java –client的模式来运行的，server即意味着你的tomcat是以真实的production的模式在运行的，这也就意味着你的tomcat以server模式运行时将拥有：更大、更高的并发处理能力，更快更强捷的JVM垃圾回收机制，可以获得更多的负载与吞吐量
* -Xms2048m -Xmx2048m

    即JVM内存设置了，把Xms与Xmx两个值设成一样是最优的做法。

    如何知道我的JVM能够使用最大值？输入命令：
    ```bat
    java  -Xmx2048m -version
    ```
    如果正确显示jdk版本，说明jvm内存可以设置到这个值，比如：

    ```
    java version "1.8.0_92"
    Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
    Java HotSpot(TM) 64-Bit Server VM (build 25.92-b14, mixed mode)
    ```

    但如果出现提示错误信息，则说明jvm内存不可以设置到这个值，比如：

    ```
    Error occurred during initialization of VM
    Unable to allocate 25922944KB bitmaps for parallel garbage collection for the requested 829534208KB heap.
    Error: Could not create the Java Virtual Machine.
    Error: A fatal exception has occurred. Program will exit.
    ```

    因此在设这个-Xms与-Xmx值时一定一定记得先这样测试一下，要不然直接加在tomcat启动命令行中你的tomcat就再也起不来了，要飞是飞不了，直接成了一只瘟猫了。
* -Xss512k

    是指设定每个线程的堆栈大小。这个就要依据你的程序，看一个线程 大约需要占用多少内存，可能会有多少线程同时运行等。一般不宜设置超过1M，要不然容易出现out ofmemory。
* -XX:PermSize=128M -XX:MaxPermSize=256M

    JVM使用-XX:PermSize设置非堆内存初始值，默认是物理内存的1/64；在数据量的很大的文件导出时，一定要把这两个值设置上，否则会出现内存溢出的错误。

    由XX:MaxPermSize设置最大非堆内存的大小，默认是物理内存的1/4。

    那么，如果是物理内存4GB，
    1/64就是64MB，这就是PermSize默认值，也就是永生代内存初始大小；
    1/4是1024MB，这就是MaxPermSize默认大小。

* -XX:+AggressiveOpts

    作用如其名（aggressive），启用这个参数，则每当JDK版本升级时，你的JVM都会使用最新加入的优化技术（如果有的话）

* -XX:+UseBiasedLocking

    启用一个优化了的线程锁，我们知道在我们的appserver，每个http请求就是一个线程，有的请求短有的请求长，就会有请求排队的现象，甚至还会出现线程阻塞，这个优化了的线程锁使得你的appserver内对线程处理自动进行最优调配。

* -XX:+DisableExplicitGC

    在程序代码中不允许有显示的调用"System.gc()"。看到过有两个极品工程中每次在DAO操作结束时手动调用System.gc()一下，觉得这样做好像能够解决它们的out ofmemory问题一样，付出的代价就是系统响应时间严重降低，就和我在关于Xms,Xmx里的解释的原理一样，这样去调用GC导致系统的JVM大起大落，性能不到什么地方去哟！

* -XX:MaxTenuringThreshold=31

    设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概率。

    这个值的设置是根据本地的jprofiler监控后得到的一个理想的值，不能一概而论原搬照抄。
* -XX:+UseConcMarkSweepGC

    即CMS gc，这一特性只有jdk1.5即后续版本才具有的功能，它使用的是gc估算触发和heap占用触发。

    我们知道频频繁的GC会造面JVM的大起大落从而影响到系统的效率，因此使用了CMS GC后可以在GC次数增多的情况下，每次GC的响应时间却很短，比如说使用了CMS GC后经过jprofiler的观察，GC被触发次数非常多，而每次GC耗时仅为几毫秒。

* -XX:+UseParNewGC

    对年轻代采用多线程并行回收，这样收得快。
* -XX:+CMSParallelRemarkEnabled

    在使用UseParNewGC 的情况下, 尽量减少 mark 的时间
* -XX:+UseCMSCompactAtFullCollection

    在使用concurrent gc 的情况下, 防止 memoryfragmention, 对live object 进行整理, 使 memory 碎片减少。
* -XX:LargePageSizeInBytes=128m

    指定 Java heap的分页页面大小
* -XX:+UseFastAccessorMethods

    get,set 方法转成本地代码
* -XX:+UseCMSInitiatingOccupancyOnly

    指示只有在 oldgeneration 在使用了初始化的比例后concurrent collector 启动收集
* -Djava.awt.headless=true

    这个参数一般我们都是放在最后使用的，这全参数的作用是这样的，有时我们会在我们的J2EE工程中使用一些图表工具如：jfreechart，用于在web网页输出GIF/JPG等流，在winodws环境下，一般我们的app server在输出图形时不会碰到什么问题，但是在linux/unix环境下经常会碰到一个exception导致你在winodws开发环境下图片显示的好好可是在linux/unix下却显示不出来，因此加上这个参数以免避这样的情况出现。

    上述这样的配置，基本上可以达到：

    * 系统响应时间增快
    * JVM回收速度增快同时又不影响系统的响应率
    * JVM内存最大化利用
    * 线程阻塞情况最小化

#### 二、Tomcat容器内的优化

tomcat线程优化是在server.xml中，如：

```xml
<Connector port="8080" protocol="HTTP/1.1"
    URIEncoding="UTF-8"  
    minSpareThreads="25" maxSpareThreads="75"
    enableLookups="false" disableUploadTimeout="true" connectionTimeout="20000"
    acceptCount="300"  maxThreads="300" maxProcessors="1000" minProcessors="5"
    useURIValidationHack="false"
    compression="on" compressionMinSize="2048" compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"
    redirectPort="8443"
/>
```

* URIEncoding="UTF-8"

    使得tomcat可以解析含有中文名的文件的url，真方便，不像apache里还有搞个mod_encoding，还要手工编译
* maxSpareThreads

    如果空闲状态的线程数多于设置的数目，则将这些线程中止，减少这个池中的线程总数。
* minSpareThreads

    最小备用线程数，tomcat启动时的初始化的线程数。
* enableLookups

    这个功效和Apache中的HostnameLookups一样，设为关闭。
* connectionTimeout

    connectionTimeout为网络连接超时时间毫秒数。
* maxThreads:

    maxThreads Tomcat使用线程来处理接收的每个请求。这个值表示Tomcat可创建的最大的线程数，即最大并发数。
* acceptCount

    acceptCount是当线程数达到maxThreads后，后续请求会被放入一个等待队列，这个acceptCount是这个队列的大小，如果这个队列也满了，就直接refuse connection

* maxProcessors与minProcessors

    在Java中线程是程序运行时的路径，是在一个程序中与其它控制线程无关的、能够独立运行的代码段。它们共享相同的地址空间。多线程帮助程序员写出CPU最大利用率的高效程序，使空闲时间保持最低，从而接受更多的请求。

    通常Windows是1000个左右，Linux是2000个左右。
* useURIValidationHack

    我们来看一下tomcat中的一段源码：
    ```java
    security
        if (connector.getUseURIValidationHack()) {
            String uri = validate(request.getRequestURI());
            if (uri == null) {
                res.setStatus(400);
                res.setMessage("Invalid URI");
                throw new IOException("Invalid URI");
            } else {
                req.requestURI().setString(uri);
                // Redoing the URI decoding
                req.decodedURI().duplicate(req.requestURI());
                req.getURLDecoder().convert(req.decodedURI(), true);
            }
        }
    ```
    可以看到如果把useURIValidationHack设成"false"，可以减少它对一些url的不必要的检查从而减省开销。
* disableUploadTimeout

    类似于Apache中的keeyalive一样

* 给Tomcat配置gzip压缩(HTTP压缩)功能

    ```xml
    compression="on" compressionMinSize="2048"
    compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"
    ```
    HTTP 压缩可以大大提高浏览网站的速度，它的原理是，在客户端请求网页后，从服务器端将网页文件压缩，再下载到客户端，由客户端的浏览器负责解压缩并浏览。相对于普通的浏览过程HTML,CSS,Javascript , Text ，它可以节省40%左右的流量。更为重要的是，它可以对动态生成的，包括CGI、PHP , JSP , ASP , Servlet,SHTML等输出的网页也能进行压缩，压缩效率惊人。

    * compression="on" 打开压缩功能

    * compressionMinSize="2048" 启用压缩的输出内容大小，这里面默认为2KB

    * noCompressionUserAgents="gozilla, traviata" 对于以下的浏览器，不启用压缩

    * compressableMimeType="text/html,text/xml"　压缩类型

    最后不要忘了把8443端口的地方也加上同样的配置，因为如果我们走https协议的话，我们将会用到8443端口这个段的配置

#### 三、使用线程池
使用线程池

在server.xml中增加executor节点，然后配置connector的executor属性，如下：

```xml
<Executorname="tomcatThreadPool" namePrefix="req-exec-"maxThreads="1000" minSpareThreads="50"maxIdleTime="60000"/>

<Connectorport="8080" protocol="HTTP/1.1"executor="tomcatThreadPool"/>
```

其中：

* namePrefix：线程池中线程的命名前缀
* maxThreads：线程池的最大线程数
* minSpareThreads：线程池的最小空闲线程数
* maxIdleTime：超过最小空闲线程数时，多的线程会等待这个时间长度，然后关闭
* threadPriority：线程优先级

注：当tomcat并发用户量大的时候，单个jvm进程确实可能打开过多的文件句柄，这时会报java.net.SocketException:Too many open files错误。可使用下面步骤检查：

* ps -ef |grep tomcat 查看tomcat的进程ID，记录ID号，假设进程ID为10001
* lsof -p 10001|wc -l 查看当前进程id为10001的 文件操作数
* ulimit -a 查看每个用户允许打开的最大文件数

#### 三种模式比较：

* BIO：一个线程处理一个请求。缺点：并发量高时，线程数较多，浪费资源。Tomcat7或以下在Linux系统中默认使用这种方式。

* NIO：利用Java的异步IO处理，可以通过少量的线程处理大量的请求。Tomcat8在Linux系统中默认使用这种方式。Tomcat7必须修改Connector配置来启动（conf/server.xml配置文件）：

```xml
<Connectorport="8080"protocol="org.apache.coyote.http11.Http11NioProtocol" connectionTimeout="20000"redirectPort="8443"/>
```

* APR(Apache Portable Runtime)：从操作系统层面解决io阻塞问题。Linux如果安装了apr和native，Tomcat直接启动就支持apr。

    * 安装apr以及tomcat-native
        ```bash
        yum -y install apr apr-devel
        ```
    * 进入tomcat/bin目录，比如：
        ```bash
        cd /opt/local/tomcat/bin/
        tar xzfv tomcat-native.tar.gz
        cd tomcat-native-1.1.32-src/jni/native
        ./configure --with-apr=/usr/bin/apr-1-config
        make && make install
        ```
    #注意最新版本的tomcat自带tomcat-native.war.gz，不过其版本相对于yum安装的apr过高，configure的时候会报错。

    解决：yum remove apr apr-devel –y,卸载yum安装的apr和apr-devel,下载最新版本的apr源码包，编译安装;或者下载低版本的tomcat-native编译安装

    安装成功后还需要对tomcat设置环境变量，方法是在catalina.sh文件中增加1行：

    ```
    CATALINA_OPTS="-Djava.library.path=/usr/local/apr/lib"
    ```

    #apr下载地址：http://apr.apache.org/download.cgi

    #tomcat-native下载地址：http://tomcat.apache.org/download-native.cgi

    修改8080端对应的conf/server.xml

    protocol="org.apache.coyote.http11.Http11AprProtocol"

    ```xml
    <Connector executor="tomcatThreadPool"
    port="8080"
    protocol="org.apache.coyote.http11.Http11AprProtocol"
    connectionTimeout="20000"
    enableLookups="false"
    redirectPort="8443"
    URIEncoding="UTF-8" />
    ```

    PS:启动以后查看日志 显示如下表示开启apr模式

    ```
    Sep 19, 2016 3:46:21 PM org.apache.coyote.AbstractProtocol start
    INFO: Starting ProtocolHandler ["http-apr-8081"]
    ```

原文：https://blog.csdn.net/lifetragedy/article/details/7708724

https://blog.csdn.net/wh211212/article/details/52732920 