### 在Linux下搭建dubbo管理控制台

在搭建dubbo管理控制台前,需要先部署一些系统环境:jdk,maven,tomcat,zookeeper

1. jdk

    参考文档：linux-jdk.md

2. maven

    参考文档：linux-maven.md

3. tomcat

    参考文档：linux-tomcat.md

4. zookeeper

    参考文档：分布式架构-分布式协调服务-zookeeper-安装.md

5. dubbo-admin

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
        cd /usr/local/apache-tomcat-8.0.53/webapps/
        rm -rf *    #删除发布目录的所有项目
        cp /usr/local/dubbo/dubbo-admin/target/dubbo-admin-2.6.0.war ./ROOT.war #copy文件
        ./bin/startup.sh    #启动tomcat，此时tomcat端口8080,dubbo-admin-2.6.0项目的zookeeper配置是127.0.0.1:2181，部署到其他环境时，要注意配置的修改
        ```

    4. 访问

        http://127.0.0.1:8080/

        默认登录账号:root/root