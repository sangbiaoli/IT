1、下载maven安装包
```bash
cd /usr/local
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
```
 
2、解压缩maven
```bash
tar -zxvf apache-maven-3.5.2-bin.tar.gz 
```

3、配置maven环境变量
```bash
vi /etc/profile

#添加环境变量
export MAVEN_HOME=/usr/local/apache-maven-3.5.2
export MAVEN_HOME
export PATH=$PATH:$MAVEN_HOME/bin
```
编辑之后记得使用source /etc/profile命令是改动生效。

5、配置setting.xml
```bash
vi /usr/local/apache-maven-3.5.2/conf/settings.xml

#在<profiles></profiles>之间添加内容
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

#开启这个标签
<activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
```

4、验证结果
```bash
mvn -version
```

5、路径
/root/.m2/repository/