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