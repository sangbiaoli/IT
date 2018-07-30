### **jdk 安装**
假如jdk-8u152-linux-x64.tar.gz在/home/bill下

1. 新建路径/usr/java,并拷贝压缩包， 并在其下解压 
    ```bash
    mkdir -p /usr/java
    cp -f /home/bill/jdk-8u152-linux-x64.tar.gz /usr/java
    cd /usr/java
    tar -zxvf jdk-8u152-linux-x64.tar.gz
    rm -f jdk-8u152-linux-x64.tar.gz
    ```
2. 添加JDK到系统环境变量 
    ```bash
    [root@test java]# vi /etc/profile 
    ```
    新增以下内容： 
    ```bash
    export JAVA_HOME=/usr/java/jdk1.8.0_152
    export PATH=$JAVA_HOME/bin:$PATH 
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
    ```
3. 使配置生效 
    ```bash
    source /etc/profile #使配置文件立即生效 
    ```
4.  查看结果：
    ```bash
    [root@test java]# java -version 
    java version "1.8.0_152"
    Java(TM) SE Runtime Environment (build 1.8.0_152-b16)
    Java HotSpot(TM) 64-Bit Server VM (build 25.152-b16, mixed mode)
    ```

### **tomcat 安装**

1. 下载apache-tomcat-8.0.53.tar.gz
    ```bash
    cd /usr/local
    wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.0.53/bin/apache-tomcat-8.0.53.tar.gz
    ```
2. 安装tomcat
    ```bash
    cd /usr/local
    tar -zxvf apache-tomcat-8.0.53.tar.gz
    ```
3. 启动tomcat
    ```
    cd /usr/local
    cd apache-tomcat-8.0.53
    ./bin/startup.sh
    ```

### **防火墙使用**
1. redhat
    ```bash
    chkconfig --level 2345 iptables off #关闭防火墙
    /etc/init.d/iptables status #查看防火墙状态
    service iptables stop    #关闭防火墙
    service iptables start   #启动防火墙
    service iptables status  #防火墙状态
    ```
    如果没有安装，可以执行以下命令安装
    ```bash
    yum install iptables
    ```

2. centos
    ```bash
    firewall-cmd --state   #查看防火墙状态。得到结果是running或者not running
    firewall-cmd --permanent --zone=public --add-port=8080/tcp #永久的添加该端口。去掉--permanent则表示临时  

    systemctl start firewalld.service #开启防火墙的命令
    systemctl stop firewalld.service #关闭防火墙的命令
    systemctl enable firewalld.service #开机自动启动
    systemctl disable firewalld.service #关闭开机自动启动
    systemctl status firewalld #查看防火墙状态
    ```
