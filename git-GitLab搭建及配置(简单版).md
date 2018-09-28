1. 安装并配置必要的依赖项
    ```bash
    yum install -y curl policycoreutils-python openssh-server
    systemctl enable sshd
    systemctl start sshd

    firewall-cmd --permanent --add-service=http
    systemctl reload firewalld
    ```
2. 添加GitLab软件包存储库并安装软件包

    官方镜像源在国外，国内安装会很慢，甚至有时因网络问题会无法安装。

    国内推荐使用清华大学开源软件镜像源。

    新建 /etc/yum.repos.d/gitlab-ce.repo，内容为：
    ```
    [gitlab-ce]
    name=Gitlab CE Repository
    baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
    gpgcheck=0
    enabled=1
    ```
    再执行
    ```bash
    yum makecache   # 更新本地YUM缓存，这个要花点时间，要耐心等下
    yum install gitlab-ce    # 自动安装最新版本
    ```

3. 修改配置文件/etc/gitlab/gitlab.rb，绑定域名

    ```bash
    external_url 'http://gitlab.xxx.com'  #如果用了虚拟机或本地安装，则改为ip,比如：http://127.0.0.1:8085
    ```
4. 启动GitLab，使得配置生效
    ```bash
    gitlab-ctl reconfigure #首次执行会很慢，执行完毕后，重启gitlab-ctl
    gitlab-ctl restart #重启
    ```
5. 测试访问

    首次登陆会跳出设置密码的界面，设置完后自动跳转到登录界面，默认用户名root。

    登陆进去后，可以更改用户名、密码等。

    初始登入时，总报502，也没有防火墙，经检查是内存不足，我是2G，官方说是最少要4G，这种情况只能增大内存。
6. 注册用户并添加到配置列表

    登录主页后，可以注册新账户
    ```bash
    $ git config --global user.name "your_username"  #设置用户名
    $ git config --global user.email "your_registered_github_Email"  #设置邮箱地址(建议用注册giuhub的邮箱)
    ```
7. SSH key

    用新的用户注册后，此时新创建项目，会报提示：
    ```
    You won't be able to pull or push project code via SSH until you add an SSH key to your profile
    ```
    因此要为该用户添加ssh key

    1. 检查是否已经有SSH Key。
        ```bash
        $cd ~/.ssh
        ```
    2. 生成一个新的SSH。
        ```bash
        $ssh-keygen -t rsa -C "注册时的user.mail"
        ```
        之后直接回车，不用填写东西。之后会让你输入密码（可以不输入密码，直接为空，这样更新代码不用每次输入 id_rsa 密码了）。然后就生成一个目录.ssh ，里面有两个文件：id_rsa , id_rsa.pub（id_rsa中保存的是私钥，id_rsa.pub中保存的是公钥）

    3. 添加ssh key到GitHub/GitLab

        在GitHub/GitLab上找到关于SSH keys->add key把id_rsa.pub公钥的完整内容复制进去就可以了。
    
    4. 最后一步测试是否成功（第一次成功后，后面都不需要再做）
        ```bash
        ssh -T git@"你的gitlab服务器ip地址"

        The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
        ECDSA key fingerprint is SHA256:RBVme/zU6Y4wlJdnK7UJunjt/3qEsihecUG+uPlvVCU.
        ECDSA key fingerprint is MD5:b3:62:58:b1:f2:78:a2:7e:12:0d:96:8f:96:06:aa:88.
        Are you sure you want to continue connecting (yes/no)? yes
        Warning: Permanently added '127.0.0.1' (ECDSA) to the list of known hosts.
        Welcome to GitLab, @XXX!
        ```
