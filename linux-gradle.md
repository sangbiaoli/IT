### **gradle 安装**

1. 下载安装包
    ```bash
    cd /usr/local/src
    wget https://downloads.gradle.org/distributions/gradle-3.2.1-all.zip
    ```
2. 解压安装
    ```bash
    unzip gradle-3.2.1-all.zip
    ```
3. 配置环境变量

    1. 打开 /etc/ 目录下的 profile 文件：
        ```bash
        vi + /etc/profile
        ```
    2. 将如下代码追加到profile 文件末尾：
        ```bash
        export GRADLE_HOME=/usr/local/src/gradle-3.2.1
        export PATH=${GRADLE_HOME}/bin:${PATH}
        ```
4. 重载/etc/profile这个文件
    ```bash
    source /etc/profile
    ```
5. 检验是否安装成功
    ```bash
    gradle -version 
    ```