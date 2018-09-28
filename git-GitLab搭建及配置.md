## GitLab搭建及配置
由于公司业务，需要上Git版本控制。

* 目前市面上比较有名的Git服务提供商，国外有GitHub、BitBucket、GitLab，国内有Coding。

* 现有的服务商，对于免费的套餐都有一定的限制。比如：
    * GitHub只允许建立免费的开源repository，建立私有的仓库需要收费。
    * BitBucket允许建立无限制的私有项目，不过对于项目中参与的开发人员是有人数限制的，当团队中开发者规模达到一定数量后，需要付费购买相应的套餐。

GitLab社区版是免费的，不但能建立免费的私有仓库而且没有数量上限，参与人员也没有数量限制，还能设置成员的权限，甚至细致到具体某条分支的权限，以及强大的工作流等等。

GitLab很适合中小型非开源项目公司。

![](git/git_GitLab.jpg)


### 一、GitLab 简介
GitLab 是一个利用Ruby on Rails 开发的开源版本控制系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。

它拥有与GitHub类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序（Wall）进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

开源中国代码托管平台 码云 就是基于GitLab项目搭建。

GitLab 分为 GitLab Community Edition(CE) 社区版 和 GitLab Enterprise Edition(EE) 专业版。社区版免费，专业版收费，两个版本在功能上的差异对比，可以参考官方对比说明

### 二、GitLab 安装和配置

安装社区版，GitLab CE 版本：9.2.6

1. **GitLab安装**

    通过GitLab官方提供的Omnibus安装包来安装，相对方便。Omnibus安装包套件整合了大部分的套件（Nginx、ruby on rails、git、redis、postgresql等），再不用额外安装这些软件，减轻了绝大部分安装量。

    GitLab官方安装文档 ：CentOS6.x系统

    1. 安装依赖包，并配置postfix服务为GitLab邮件服务
    ```bash
    yum install curl openssh-server openssh-clients postfix cronie
    service postfix start
    chkconfig postfix on

    # 若不执行安装有可能遇到这个错误【lokkit: command not found 】
    yum -y install lokkit
    lokkit -s http -s ssh
    ```
    2. 打开HTTP和SSH端口
    ```bash
    iptables -I INPUT -m tcp -p tcp --dport 22 -j ACCEPT
    iptables -I INPUT -m tcp -p tcp --dport 80 -j ACCEPT
    ```
    3. 两种安装源
        * 从官方镜像源安装

            添加GitLab仓库并安装到服务器上
            ```bash
            curl -sS http://packages.gitlab.cc/install/gitlab-ce/script.rpm.sh | sudo bash
            yum install gitlab-ce     # 自动安装最新版本
            yum install gitlab-ce-9.2.1-ce.0.el6     # 安装指定版本
            ```
        * 从第三方镜像源安装

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
    4. 修改配置文件/etc/gitlab/gitlab.rb，绑定域名

        ```bash
        external_url 'http://gitlab.xxx.com'  #如果用了虚拟机或本地安装，则改为ip,比如：http://127.0.0.1
        ```
    5. 启动GitLab，使得配置生效
        ```bash
         gitlab-ctl reconfigure
        ```

    6. 在Dnspod中添加解析记录
    7. 使用浏览器访问GitLab

        > 首次访问GitLab,系统会让你重新设置管理员的密码,设置成功后会返回登录界面.

        > 默认的管理员账号是root,如果你想更改默认管理员账号,请输入上面设置的新密码登录系统后修改帐号名.

    8. GitLab安装细节
        * 主配置文件: /etc/gitlab/gitlab.rb
        * GitLab 文档根目录: /opt/gitlab
        * 默认存储库位置: /var/opt/gitlab/git-data/repositories
        * GitLab Nginx 配置文件路径:  /var/opt/gitlab/nginx/conf/gitlab-http.conf
        * Postgresql 数据目录: /var/opt/gitlab/postgresql/data
    9. GitLab由以下服务构成
        * nginx: 静态web服务器
        * gitlab-shell: 用于处理Git命令和修改authorized keys列表
        * gitlab-workhorse: 轻量级的反向代理服务器
        * logrotate：日志文件管理工具
        * postgresql：数据库
        * redis：缓存数据库
        * sidekiq：用于在后台执行队列任务（异步执行）
        * unicorn：An HTTP server for Rack applications，GitLab Rails应用是托管在这个服务器上面的。

2. **配置SMTP服务**

    如果你不想用服务器自带的postfix服务来发邮件，可以改用SMTP服务。

    修改GitLab邮件服务配置(gitlab.rb文件)，使用腾讯企业邮箱的SMTP服务器，填写账号和密码
    ```
    gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
    gitlab_rails['smtp_port'] = 25
    gitlab_rails['smtp_user_name'] = "xxx"
    gitlab_rails['smtp_password'] = "xxx"
    gitlab_rails['smtp_domain'] = "smtp.qq.com"
    gitlab_rails['smtp_authentication'] = 'plain'
    gitlab_rails['smtp_enable_starttls_auto'] = true
    ```
    使配置生效
    ```bash
     gitlab-ctl reconfigure
     gitlab-rake cache:clear RAILS_ENV=production      # 清除缓存
    ```
3. **GitLab配置HTTPS**

    GitLab默认是使用HTTP的，可以手动配置为HTTPS

    1. 上传SSL证书
        创建ssl目录，用于存放SSL证书
        ```bash
        # mkdir -p /etc/gitlab/ssl
        # chmod 0700 /etc/gitlab/ssl
        ```
        上传证书并修改证书权限
        ```bash
        # chmod 600 /etc/gitlab/ssl/*
        ```
    2. 修改GitLab的配置文件

        修改配置文件/etc/gitlab/gitlab.rb
        ```bash
        external_url "https://gitlab.xxx.com"
        nginx['redirect_http_to_https'] = true
        nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.xxx.com.crt"
        nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.xxx.com.key"
        ```
        重建配置，使其生效
        ```bash
        # gitlab-ctl reconfigure
        ```
        以上操作后，GitLab自带的Nginx服务的配置文件 /var/opt/gitlab/nginx/conf/gitlab-http.conf 会被重新修改：
        ```bash
        server {
        listen *:80;
        server_name gitlab.xxx.com;
        server_tokens off; ## Don't show the nginx version number, a security best practice
        return 301 https://gitlab.xxx.com:443$request_uri;
        access_log  /var/log/gitlab/nginx/gitlab_access.log gitlab_access;
        error_log   /var/log/gitlab/nginx/gitlab_error.log;
        }
        ```
        不用额外再配置，HTTP 会自动跳转到 HTTPS 。

    3. 开放443端口

        在防火墙上开放443端口，用于HTTPS
        ```bash
        # iptables -I INPUT -m tcp -p tcp --dport 443 -j ACCEPT
        ```
4. **修改root用户密码**

    对于普通用户而言，可通过系统重置密码，接收邮件即可。可是GitLab管理员账号，缺省邮箱 admin@example.com 是个不存在的邮箱地址，无法通过邮箱修改密码。
    官方修改密码文档，根据文档，修改root密码的方法如下：

    1. 打开与Rails程序交互的控制台
        在root权限下，执行：
        ```bash
        # gitlab-rails console production
         ```
        等待一会，直到控制台加载成功。
    2. 获取用户信息并修改root用户密码
        ```bash
        # gitlab-rails console production
        Loading production environment (Rails 4.2.8)
        irb(main):001:0> user = User.where(id: 1).first
        => #<User id: 1, email: "admin@example.com"......
        irb(main):009:0> user.password = '12345678'
        => "12345678"
        irb(main):010:0> user.password_confirmation = '12345678'
        => "12345678"
        irb(main):011:0> user.save!
        Enqueued ActionMailer::DeliveryJob (Job ID: 510bb5be-a156-4522-9983-44d8a895e92a) to Sidekiq(mailers) with arguments: "DeviseMailer", "password_change", "deliver_now", gid://gitlab/User/1
        => true
        irb(main):011:0> exit
        ```
### 三、GitLab 常用命令
1. 运维管理排查
    ```bash
    # 查看版本
    cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
    # 检查gitlab
    gitlab-rake gitlab:check SANITIZE=true --trace
    # 实时查看日志
    gitlab-ctl tail
    # 数据库关系升级
    gitlab-rake db:migrate
    # 清理redis缓存
    gitlab-rake cache:clear
    # 升级GitLab-ce 版本
    yum update gitlab-ce
    # 升级PostgreSQL最新版本
    gitlab-ctl pg-upgrade
    ```
2. 服务管理
    ```bash
    # 启动所有 gitlab 组件：
    gitlab-ctl start
    # 停止所有 gitlab 组件：
    gitlab-ctl stop
    # 停止所有 gitlab postgresql 组件：
    gitlab-ctl stop postgresql
    # 停止相关数据连接服务
    gitlab-ctl stop unicorn
    gitlab-ctl stop sidekiq
    # 重启所有 gitlab 组件：
    gitlab-ctl restart
    # 重启所有 gitlab gitlab-workhorse 组件：
    gitlab-ctl restart  gitlab-workhorse
    # 查看服务状态
    gitlab-ctl status
    # 生成配置并启动服务
    gitlab-ctl reconfigure
    ```
3. 日志
    ```bash
    # 实时查看所有日志
    gitlab-ctl tail
    # 实时检查redis的日志
    gitlab-ctl tail redis
    
    # 实时检查postgresql的日志
    gitlab-ctl tail postgresql
    
    # 检查gitlab-workhorse的日志
    gitlab-ctl tail gitlab-workhorse
    
    # 检查logrotate的日志
    gitlab-ctl tail logrotate
    
    # 检查nginx的日志
    gitlab-ctl tail nginx
    
    # 检查sidekiq的日志
    gitlab-ctl tail sidekiq
    
    # 检查unicorn的日志
    gitlab-ctl tail unicorn
    ```
### 四、GitLab备份和恢复
1. 备份

    GitLab作为公司项目代码的版本管理系统，数据非常重要，必须做好备份。

2. 修改备份目录

    GitLab备份的默认目录是 /var/opt/gitlab/backups ，如果想改备份目录，可修改/etc/gitlab/gitlab.rb：
    ```bash
    gitlab_rails['backup_path'] = '/data/backups'
    ```
    修改配置后，记得：
    ```bash
    gitlab-ctl reconfigure
    ```
3. 备份命令
    ```bash
    gitlab-rake gitlab:backup:create
    ```
    该命令会在备份目录（默认：/var/opt/gitlab/backups/）下创建一个tar压缩包xxxxxxxx_gitlab_backup.tar，其中开头的xxxxxx是备份创建的时间戳，这个压缩包包括GitLab整个的完整部分。

4. 自动备份

    通过任务计划crontab 实现自动备份

    ```bash
    # 每天2点备份gitlab数据
    0 2 * * * /usr/bin/gitlab-rake gitlab:backup:create
    ```
5. 备份保留7天

    可设置只保留最近7天的备份，编辑配置文件 /etc/gitlab/gitlab.rb
    ```bash
    # 数值单位：秒
    gitlab_rails['backup_keep_time'] = 604800
    ```
    重新加载gitlab配置文件
    ```bash
    gitlab-ctl reconfigure
    ```
5. 恢复

    备份文件：
    ```bash
    /var/opt/gitlab/backups/1499244722_2017_07_05_9.2.
    6_gitlab_backup.tar
    ```

    停止 unicorn 和 sidekiq ，保证数据库没有新的连接，不会有写数据情况。
    ```bash
    # 停止相关数据连接服务
    gitlab-ctl stop unicorn
    gitlab-ctl stop sidekiq
    # 指定恢复文件，会自动去备份目录找。确保备份目录中有这个文件。
    # 指定文件名的格式类似：1499242399_2017_07_05_9.2.6，程序会自动在文件名后补上：“_gitlab_backup.tar”
    # 一定按这样的格式指定，否则会出现 The backup file does not exist! 的错误
    gitlab-rake gitlab:backup:restore BACKUP=1499242399_2017_07_05_9.2.6
    # 启动Gitlab
    gitlab-ctl start
    ```

参考：http://www.hjqjk.com/2017/GitLab-install-config.html