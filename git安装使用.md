## git安装使用

1. 安装Git
    ```bash
    yum -y install git
    ```
2. 验证Git是否安装成功
    ```bash
    git --version
    ```
3. 添加用户Git
    ```bash
    sudo useradd -r -s /bin/sh -c 'git version control' -d /home/git git
    ```
4. 设置权限
    ```bash
    mkdir -p /home/git
    chown git:git /home/git
    ```

参考：https://blog.csdn.net/a214919447/article/details/55259997