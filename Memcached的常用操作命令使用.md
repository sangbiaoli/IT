## memcached 常用命令及使用说明
1. 启动Memcache 常用参数
    * -p <num>      设置TCP端口号(默认设置为: 11211)
    * -U <num>      UDP监听端口(默认: 11211, 0 时关闭) 
    * -l <ip_addr>  绑定地址(默认:所有都允许,无论内外网或者本机更换IP，有安全隐患，若设置为127.0.0.1就只能本机访问)
    * -c <num>      max simultaneous connections (default: 1024)
    * -d            以daemon方式运行
    * -u <username> 绑定使用指定用于运行进程<username>
    * -m <num>      允许最大内存用量，单位M (默认: 64 MB)
    * -P <file>     将PID写入文件<file>，这样可以使得后边进行快速进程终止, 需要与-d 一起使用

    更多可以使用者 memcached -h

2. 连接和退出
    ```bash
    telnet 127.0.0.1 11211
    quit
    ```
3. 基本命令

    五种基本 memcached 命令执行最简单的操作。这些命令和操作包括：

    * set
    * add
    * replace
    * get
    * delete
    > 前三个命令是用于操作存储在 memcached 中的键值对的标准修改命令。它们都非常简单易用，且都使用如下所示的语法：
    ```bash
    command <key> <flags> <expiration time> <bytes>
    <value>
    ```
    参数说明如下：
    * command set/add/replace
    * key     key 用于查找缓存值
    * flags     可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息
    * expiration time     在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）
    * bytes     在缓存中存储的字节点
    * value     存储的值（始终位于第二行）
    复制代码
    现在，我们来看看这些命令的实际使用。

    **3.1 set**
    > set 命令用于向缓存添加新的键值对。如果键已经存在，则之前的值将被替换。
    注意以下交互，它使用了 set 命令：
    ```bash
    set userId 0 0 5
    12345
    STORED
    ```
    > 如果使用 set 命令正确设定了键值对，服务器将使用单词 STORED 进行响应。本示例向缓存中添加了一个键值对，其键为userId，其值为12345。并将过期时间设置为 0，这将向 memcached 通知您希望将此值存储在缓存中直到删除它为止。

    **3.2 add**
    > 仅当缓存中不存在键时，add 命令才会向缓存中添加一个键值对。如果缓存中已经存在键，则之前的值将仍然保持相同，并且您将获得响应 NOT_STORED。
    下面是使用 add 命令的标准交互：
    ```bash
    set userId 0 0 5
    12345
    STORED
    add userId 0 0 5
    55555
    NOT_STORED
    add companyId 0 0 3
    564
    STORED
    ```
    **3.3 replace**
    > 仅当键已经存在时，replace 命令才会替换缓存中的键。如果缓存中不存在键，那么您将从 memcached 服务器接受到一条 NOT_STORED 响应。
    下面是使用 replace 命令的标准交互：
    ```bash
    replace accountId 0 0 5
    67890
    NOT_STORED
    set accountId 0 0 5
    67890
    STORED
    replace accountId 0 0 5
    55555
    STORED
    ```
    > 最后两个基本命令是 get 和 delete。这些命令相当容易理解，并且使用了类似的语法，如下所示：
    ```bash
    command <key>
    ```
    接下来看这些命令的应用。

    **3.4 get**
    get 命令用于检索与之前添加的键值对相关的值。您将使用 get 执行大多数检索操作。
    下面是使用 get 命令的典型交互：
    ```bash
    set userId 0 0 5
    12345
    STORED
    get userId
    VALUE userId 0 5
    12345
    END
    get bob
    END
    ```
    如您所见，get 命令相当简单。您使用一个键来调用 get，如果这个键存在于缓存中，则返回相应的值。如果不存在，则不返回任何内容。

    **3.5 delete**
    > 最后一个基本命令是 delete。delete 命令用于删除 memcached 中的任何现有值。您将使用一个键调用delete，如果该键存在于缓存中，则删除该值。如果不存在，则返回一条NOT_FOUND 消息。
    下面是使用 delete 命令的客户机服务器交互：
    ```bash
    set userId 0 0 5
    98765
    STORED
    delete bob
    NOT_FOUND
    delete userId
    DELETED
    get userId
    END
    ```
    > 可以在 memcached 中使用的两个高级命令是 gets 和 cas。gets 和cas 命令需要结合使用。您将使用这两个命令来确保不会将现有的名称/值对设置为新值（如果该值已经更新过）。我们来分别看看这些命令。

    **3.6 gets**
    > gets 命令的功能类似于基本的 get 命令。两个命令之间的差异在于，gets 返回的信息稍微多一些：64 位的整型值非常像名称/值对的 “版本” 标识符。下面是使用 gets 命令的客户机服务器交互：
    ```bash
    set userId 0 0 5
    12345
    STORED
    get userId
    VALUE userId 0 5
    12345
    END
    gets userId
    VALUE userId 0 5 4
    12345
    END
    ```
    > 考虑 get 和 gets 命令之间的差异。gets 命令将返回一个额外的值 — 在本例中是整型值 4，用于标识名称/值对。如果对此名称/值对执行另一个set 命令，则gets 返回的额外值将会发生更改，以表明名称/值对已经被更新。显示了一个例子：
    ```bash
    set userId 0 0 5
    33333
    STORED
    gets userId
    VALUE userId 0 5 5
    33333
    END
    ```
    您看到 gets 返回的值了吗？它已经更新为 5。您每次修改名称/值对时，该值都会发生更改。

    **3.7 cas**
    > cas（check 和 set）是一个非常便捷的 memcached 命令，用于设置名称/值对的值（如果该名称/值对在您上次执行 gets 后没有更新过）。它使用与 set 命令相类似的语法，但包括一个额外的值：gets 返回的额外值。
    注意以下使用 cas 命令的交互：
    ```bash
    set userId 0 0 5
    55555
    STORED
    gets userId
    VALUE userId 0 5 6
    55555
    END
    cas userId 0 0 5 6
    33333
    STORED
    ```
    > 如您所见，我使用额外的整型值 6 来调用 gets 命令，并且操作运行非常顺序。现在，我们来看看中的一系列命令：
    使用旧版本指示符的 cas 命令
    ```bash
    set userId 0 0 5
    55555
    STORED
    gets userId
    VALUE userId 0 5 8
    55555
    END
    cas userId 0 0 5 6
    33333
    EXISTS
    ```