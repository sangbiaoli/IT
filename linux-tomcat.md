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

3.  tomcat-users.xml用户配置

    在标签<tomcat-users>添加角色用户

    ```xml
    <tomcat-users>
        <role rolename="manager-gui"/>
        <role rolename="manager-script"/>
        <user username="tomcat" password="tomcat" roles="manager-gui"/>
        <user username="admin" password="123456" roles="manager-script"/>
    </tomcat-users>
    ```

4. 启动tomcat

    ```bash
    cd /usr/local/src
    cd apache-tomcat-8.0.53
    ./bin/startup.sh
    ```

    注意防火墙,参考linux-常用命令.md的防火墙部分.