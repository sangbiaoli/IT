### **linux替换yum**
redhat默认自带的yum源需要注册，才能更新，报错：

This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.

可替换为centos对应的源。 操作如下：


1. 检查是否安装yum包。查看RHEL是否安装了yum，若是安装了，那么又有哪些yum包：
    ```bash
    rpm -qa |grep yum
    ```
2. 删除redhat自带的yum包
    ```bash
    rpm -qa|grep yum|xargs rpm -e --nodeps #不检查依赖，直接删除rpm包
    rpm -qa |grep yum #查询确认
    ```
3. 下载新的yum包。使用Centos6.5的yum包
    1） 查看版本号和系统类别：

        cat /etc/redhat-release
        arch

    2）根据上一步，找到对应的yum包，然后下载。我的服务器对应的为：
        
        wget http://mirrors.163.com/centos/6/ ... 2-16.el6.x86_64.rpm
        wget http://mirrors.163.com/centos/6/ ... 6.centos.noarch.rpm
        wget http://mirrors.163.com/centos/6/ ... 0-30.el6.noarch.rpm
    
        可以通过http://mirrors.163.com/centos下载，这是笔者已经下载好的http://pan.baidu.com/s/1qW0MbgC
    3）执行：

        rpm -ivh yum-metadata-parser-1.1.2-16.el6.x86_64.rpm yum-3.2.29-40.el6.centos.noarch.rpm yum-plugin-fastestmirror-1.1.30-14.el6.noarch.rpm

        如果这一步报错，大底如下：
            libc.so.6 is needed by yum-metadata-parser-1.1.2-16.el6.i686
            libc.so.6(GLIBC_2.0) is needed by yum-metadata-parser-1.1.2-16.el6.i686
            libc.so.6(GLIBC_2.1.3) is needed by yum-metadata-parser-1.1.2-16.el6.i686
            libglib-2.0.so.0 is needed by yum-metadata-parser-1.1.2-16.el6.i686
            libpthread.so.0 is needed by yum-metadata-parser-1.1.2-16.el6.i686
            libpython2.6.so.1.0 is needed by yum-metadata-parser-1.1.2-16.el6.i686
            libsqlite3.so.0 is needed by yum-metadata-parser-1.1.2-16.el6.i686
            libxml2.so.2 is needed by yum-metadata-parser-1.1.2-16.el6.i686
            libxml2.so.2(LIBXML2_2.4.30) is needed by yum-metadata-parser-1.1.2-16.el6.i686
        则说明下载安装的yum包与系统版本不匹配，需要重新下载安装。另外，163镜像站点对于6.0~6.5的资源均合并在6的目录下。
        
    4）更换yum源，将原有源删除或备份到别的目下下：
        cd /etc/yum.repos.d/
        wget  http://mirrors.163.com/.help/CentOS6-Base-163.repo
        vi CentOS6-Base-163.repo
        编辑文件，把文件里面的$releasever全部替换为版本号：6（注意，不是6.5！）最后保存！
        为了防止错误，也可使用我已修改好的文件http://pan.baidu.com/s/1o6AZ23o

4. 清除原有缓存，重建缓存：
    clean all
    yum makecache
    
5. 更新系统：
    yum update