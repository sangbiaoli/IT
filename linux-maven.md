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