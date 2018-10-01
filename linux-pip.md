
## python安装pip出现No package python-pip available

安装pip：

1. 使用yum进行安装

    ```bash
    yum install python-pip
    ```
    若出现
    ```bash
    No package python-pip available.
    ```

    则解决方法如下：
    ```bash
    yum -y install epel-release
    yum install python-pip
    ```

2. 安装完成后清理
    ```bash
    yum clean all
    ```

3. pip验证
    ```bash
    # pip -V
    pip 8.1.2 from /usr/lib/python2.7/site-packages (python 2.7)
    ```
参考：https://blog.csdn.net/u011418530/article/details/79986251