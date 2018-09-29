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
### **gradle 安装**

1. 下载安装包
    ```bash
    cd /usr/local/src
    wget https://downloads.gradle.org/distributions/gradle-3.2.1-all.zip
    ```
2. 解压安装
    ```bash
    unzip gradle-3.2.1-all.zip
    ```
3. 配置环境变量

    1. 打开 /etc/ 目录下的 profile 文件：
        ```bash
        vi + /etc/profile
        ```
    2. 将如下代码追加到profile 文件末尾：
        ```bash
        export GRADLE_HOME=/usr/local/src/gradle-3.2.1
        export PATH=${GRADLE_HOME}/bin:${PATH}
        ```
4. 重载/etc/profile这个文件
    ```bash
    source /etc/profile
    ```
5. 检验是否安装成功
    ```bash
    gradle -version 
    ```
### **ant 安装**

1. 下载安装包
    ```bash
    cd /usr/local/src
    wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.7-bin.tar.gz
    ```
2. 解压安装
    ```bash
    tar -zxvf apache-ant-1.9.7-bin.tar.gz
    ```
3. 配置环境变量

    1. 打开 /etc/ 目录下的 profile 文件：
        ```bash
        vi + /etc/profile
        ```
    2. 将如下代码追加到profile 文件末尾：
        ```bash
        export ANT_HOME=/usr/local/src/apache-ant-1.9.7
        export PATH=${ANT_HOME}/bin:${PATH}
        ```
4. 重载/etc/profile这个文件
    ```bash
    source /etc/profile
    ```
5. 检验是否安装成功
    ```bash
    ant -version 
    ```
### **maven 安装**

1. 下载安装包
    ```bash
    cd /usr/local/src
    wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz    
    ```
2. 解压安装
    ```bash
    tar -zxvf apache-maven-3.5.2-bin.tar.gz 
    ```
3. 配置环境变量

    1. 打开 /etc/ 目录下的 profile 文件：
        ```bash
        vi + /etc/profile
        ```
    2. 将如下代码追加到profile 文件末尾：
        ```bash
        export MAVEN_HOME=/usr/local/src/apache-maven-3.5.2
        export MAVEN_HOME
        export PATH=${MAVEN_HOME}/bin:${PATH}
        ```
4. 重载/etc/profile这个文件
    ```bash
    source /etc/profile
    ```
5. 检验是否安装成功
    ```bash
    mvn -version
    ```
6. 配置setting.xml
    ```bash
    vi /usr/local/apache-maven-3.5.2/conf/settings.xml
    ```
    在<profiles></profiles>之间添加内容
    ```xml
    <profile> 
        <id>mvnrepository</id> 
        <repositories> 
        <repository> 
            <id>mvnrepository</id> 
            <name>mvnrepository</name> 
            <url>http://mvnrepository.com/</url> 
            <releases>  
                <enabled>true</enabled>  
            </releases>  
            <snapshots>  
                <enabled>false</enabled>  
            </snapshots> 
        </repository> 
        </repositories> 
    </profile>
    <profile> 
        <id>searchmvnrepository</id> 
        <repositories> 
        <repository> 
            <id>searchmvnrepository</id> 
            <name>searchmvnrepository</name> 
            <url>http://search.maven.org</url> 
            <releases>  
                <enabled>true</enabled>  
            </releases>  
            <snapshots>  
                <enabled>false</enabled>  
            </snapshots> 
        </repository> 
        </repositories> 
    </profile>   
    ```
    开启这个标签
    ```xml
    <activeProfiles>
        <activeProfile>alwaysActiveProfile</activeProfile>
        <activeProfile>anotherAlwaysActiveProfile</activeProfile>
    </activeProfiles>
    ```
6. 路径
    /root/.m2/repository/
    
### **tomcat 安装**

1. 下载apache-tomcat-8.0.53.tar.gz
    ```bash
    cd /usr/local/src
    wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.0.53/bin/apache-tomcat-8.0.53.tar.gz
    ```
2. 安装tomcat
    ```bash
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
