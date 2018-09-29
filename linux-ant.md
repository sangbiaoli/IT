### **ant 安装**

1. 下载安装包
    ```bash
    cd /usr/local/src
    wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.7-bin.tar.gz
    ```
2. 解压安装
    ```bash
    tar -zxvf apache-ant-1.9.7-bin.tar.gz
    ```
3. 配置环境变量

    1. 打开 /etc/ 目录下的 profile 文件：
        ```bash
        vi + /etc/profile
        ```
    2. 将如下代码追加到profile 文件末尾：
        ```bash
        export ANT_HOME=/usr/local/src/apache-ant-1.9.7
        export PATH=${ANT_HOME}/bin:${PATH}
        ```
4. 重载/etc/profile这个文件
    ```bash
    source /etc/profile
    ```
5. 检验是否安装成功
    ```bash
    ant -version 
    ```