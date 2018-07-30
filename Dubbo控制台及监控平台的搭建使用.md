### 在Linux下搭建dubbo管理控制台
一共需要安装四个部分：jdk，zookeeper，tomcat，dubbo-admin，已安装的部分可以略过

**1、安装JDK**

参考文档：linux常见软件安装.md，安装jdk部分

**2、安装zookeeper**

参考文档：Zookeepe介绍&安装配置&常用命令使用.md，安装zookeeper部分

**3、 安装tomcat**

参考文档：linux常见软件安装.md，安装tomcat部分

**4、 安装maven**

参考文档：Maven安装.md

**5、 安装dubbo-admin**
1. 下载源码
    ```bash
    git clone https://github.com/alibaba/dubbo.git
    git checkout -b dubbo-2.5.6
    ```
2. 编译打包
    ```bash
    cd dubbo
    mvn install -Dmaven.test.skip=true  #这个编译要花蛮长时间，因为要下载好多jar包
    cd /usr/local/dubbo/dubbo-admin/target
    ll  #可以看到有dubbo-admin-2.6.0.war，这是最终想要的文件
    ```
3. tomcat启动
    ```bash
    cd /usr/local/apache-tomcat-8.0.53
    cp /usr/local/dubbo/dubbo-admin/target/dubbo-admin-2.6.0.war ./ #copy文件
    ./bin/startup.sh    #启动tomcat，此时tomcat端口8080,dubbo-admin-2.6.0项目的dubbo配置是127.0.0.1:2181，部署到其他环境时，要注意配置的修改
    ```
