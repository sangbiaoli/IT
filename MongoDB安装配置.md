## 安装配置及常用命令

1. 官网下载MongoDB，版本选择RHEL7 Linux 64-bit x64

    https://www.mongodb.com/download-center?ct=false#community

    ```bash
    cd /usr/local/src
    wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.4.9.tgz
    ```

2. 解压缩
    ```bash
    cd /usr/local/src
    tar -zxvf mongodb-linux-x86_64-rhel70-3.4.9.tgz
    mv mongodb-linux-x86_64-rhel70-3.4.9 mongodb
    ```

3. 创建mongodb.conf文件
    ```bash
    cd /usr/local/src/mongodb/bin
    vi mongodb.conf
    ```
    输入以下内容：
    ```
    dbpath=/usr/local/src/mongodb/data/db
    logpath=/usr/local/src/mongodb/data/logs/mongodb.log
    logappend=true
    fork=true
    port=27017
    nohttpinterface = true
    #auth=true
    ```

4. 在mongodb目录下创建文件夹
    ```bash
    mkdir data
    cd data
    mkdir db
    mkdir logs
    ```
5. 运行MongoDB
    ```bash
    cd /usr/local/src/mongodb/bin
    ./mongod -f mongodb.conf
    ```
6. 测试是否运行成功

    打开浏览器输入 http://127.0.0.1:27017

    It looks like you are trying to access MongoDB over HTTP on the native driver port.
    证明安装成功


参考：https://blog.csdn.net/mawenwu1983/article/details/78139772 
