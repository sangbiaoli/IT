
### **防火墙使用**
1. redhat
    ```bash
    chkconfig --level 2345 iptables off #关闭防火墙
    /etc/init.d/iptables status #查看防火墙状态
    service iptables stop    #关闭防火墙
    service iptables start   #启动防火墙
    service iptables status  #防火墙状态
    ```
    如果没有安装，可以执行以下命令安装
    ```bash
    yum install iptables
    ```

2. centos
    ```bash
    firewall-cmd --state   #查看防火墙状态。得到结果是running或者not running
    firewall-cmd --permanent --zone=public --add-port=8080/tcp #永久的添加该端口。去掉--permanent则表示临时  

    systemctl start firewalld.service #开启防火墙的命令
    systemctl stop firewalld.service #关闭防火墙的命令
    systemctl enable firewalld.service #开机自动启动
    systemctl disable firewalld.service #关闭开机自动启动
    systemctl status firewalld #查看防火墙状态
    ```

    * 停止端口
    
    ```bash
    #bash
    port=$1
    echo "start to stop $port..."

    pid=$(netstat -nlp | grep :$port | awk '{print $7}' | awk -F"/" '{ print $1 }');

    if [  -n  "$pid"  ];  then
        kill  -9  $pid;
    fi

    echo "stoped $port ..."
    ```

    ```bash
    chmod a+x stopPort.sh
    ```
