## 基本操作

1. 建立管理员用户
    ```bash
    cd /usr/local/src/mongodb/bin
    ./mongo
    use admin
    db.createUser(
        {
            user: "dba",
            pwd: "123456",
            roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
        }
    )
    ```
2. 建立数据库

    需要插入一条记录，不然数据库无法建立
    ```bash
    use web
    db.items.insert({"name":"web"})
    ```
3. 给新创建的库加入用户
    ```bash
    use web
    db.createUser(
        {
            user: "web",
            pwd: "123456",
            roles: [ { role: "readWrite", db: "web" } ]
        }
    )
    ```
4. 关闭数据库
    ```bash
    use admin
    db.shutdownServer()
    ```
5. 给MongoDB加入验证
    ```bash
    vi mongodb.conf
    #auth=true去掉注释
    ```
6. 重新运行MongoDB
    ```bash
    ./mongod -f mongodb.conf
    ```
7. 常用操作命令
    * 登录数据库
        ```bash
        use 数据库名
        db.auth("用户名","密码")
        ```
    * 显示所有数据库
        ```bash
        show dbs
        ```
    * 显示用户
        ```bash
        use 数据库名
        show users
        ```
    * 选择数据库
        ```bash
        use 数据库名
        ```
    * 删除数据库
        ```
        use 数据库名
        db.dropDatabase()
        ```
    * 删除用户
        ```bash
        use web
        db.system.users.remove({user:"用户名"})
        ```
    * 关闭数据库
        ```bash
        use admin
        db.shutdownServer()
        ```
7. 备份恢复数据库
    * 备份
        ```bash
        mongodump -d <数据库名> <文件夹目录>
        ```
    * 恢复
        ```bash
        mongorestore -d <数据库名> <文件夹目录>
        ```
8. MongoDB参数解释
    * --dbpath 数据库路径(数据文件)
    * --logpath 日志文件路径
    * --master 指定为主机器
    * --slave 指定为从机器
    * --source 指定主机器的IP地址
    * --pologSize 指定日志文件大小不超过64M.因为resync是非常操作量大且耗时，最好通过设置一个足够大的oplogSize来避免resync(默认的 oplog大小是空闲磁盘大小的5%)。
    * --logappend 日志文件末尾添加，即使用追加的方式写日志
    * --journal 启用日志
    * --port 启用端口号
    * --fork 在后台运行
    * --only 指定只复制哪一个数据库
    * --slavedelay 指从复制检测的时间间隔
    * --auth 是否需要验证权限登录(用户名和密码)
    * --syncdelay 数据写入硬盘的时间（秒），0是不等待，直接写入
    * --notablescan 不允许表扫描
    * --maxConns 最大的并发连接数，默认2000
    * --pidfilepath 指定进程文件，不指定则不产生进程文件
    * --bind_ip 绑定IP，绑定后只能绑定的IP访问服务


参考：https://blog.csdn.net/mawenwu1983/article/details/78139772 
