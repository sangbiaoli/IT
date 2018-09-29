### **jdk 安装**
假如jdk-8u152-linux-x64.tar.gz在/usr/local/src下

1. 新建路径/usr/java,并拷贝压缩包， 并在其下解压 
    ```bash
    mkdir -p /usr/java
    cp -f /usr/local/src/jdk-8u152-linux-x64.tar.gz /usr/java
    cd /usr/java
    tar -zxvf jdk-8u152-linux-x64.tar.gz
    rm -f jdk-8u152-linux-x64.tar.gz
    ```
2. 添加JDK到系统环境变量 
    ```bash
    [root@test java]# vi + /etc/profile 
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