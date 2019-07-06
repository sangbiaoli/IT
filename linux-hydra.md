## hydra爆破工具

1. 下载

    网上官网地址不能下载了，因此用本地的包 hydra-7.1-src.tar.gz。

2. 安装依赖包

    * 如果是Debian和Ubuntu发行版，yum源里自带hydra，我们直接用apt-get在线安装。

        ```bash
        # apt-get -y install
        libssl-dev libssh-dev libidn11-dev libpcre3-dev
        libgtk2.0-dev libmysqlclient-dev libpq-dev libsvn-dev
        firebird2.1-dev libncp-dev hydra gcc
        ```

    * 对于Redhat/Centos/Fedora发行版，我们需要下载源码包，然后编译安装，因此先安装相关依赖包

        ```bash
        # yum -y install
        openssl-devel  pcre-devel  ncpfs-devel postgresql-devel
        libssh-devel  subversion-devel  gcc
        ```

3. 编译安装hydra

    ```bash
    # tar zxvf hydra-7.1.tar.gz
    # cd hydra-7.1
    # ./configure --prefix=/path/to/hydra
    # make && make install
    ```bash

4. 使用方法

    1. 要先下载字典爆破，链接：https://download.csdn.net/download/av85461883/4340522

    2. 加载后解压字典，把user.txt和password.txt放到某个目录，比如/usr/local/src

    3. 比如要爆破ftp，则执行命令

        ```bash
        hydra 192.71.181.185 ftp -l user.txt -P password.txt -t 16 -vV
        ```


参考：https://blog.csdn.net/yjk13703623757/article/details/53216501 