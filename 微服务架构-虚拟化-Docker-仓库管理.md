## Docker仓库管理

1. 仓库概述

    * 仓库（Repository）：Docker仓库主要用于镜像的存储，它是镜像分发. 部署的关键。仓库分为公共仓库和私有仓库。
    * 注册服务器(Registry)和仓库区别：注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）
    * 官方的公用仓库Docker Hub：如果仅仅是搜索和使用Docker Hub的公共镜像，不需要Docker Hub账户就可以直接操作。如果要上传和分享我们自己创建的镜像，就需要Docker Hub账户。注：注册账户需要借助FQ工具

2. 仓库管理
    1. 注册账号
        https://hub.docker.com/ #在此页面注册账号，需要用户名，邮箱，密码（注:需要FQ才能注册，注册通过邮箱激活后可以通过网页登陆）
    2. 登陆docker hub
        ```bash
        root@localhost ~]# docker login
        #Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
        Username: *******      
        Password: 
        Login Succeeded
        ```
    3. 查找镜像   
        ```bash
        #可参考https://www.cnblogs.com/yangleitao/p/9683104.html
        [root@localhost ~]# docker search centos #可以加上版本号
        ```
    4. 下载镜像
        ```bash
        [root@localhost ~]# docker pull centos
        ```
    5. 上传镜像
        ```bash
        #我们可以把自己的镜像传到docker hub官网上，前提是已经注册了账号
        [root@localhost ~]# docker push image_name
        ```
3. 搭建私有仓库

    registy为docker官方提供的一个镜像，我们可以用它来创建本地的docker私有仓库。
    ```bash
    #下载registry 镜像
    [root@localhost ~]# docker pull registry

    #启动镜像
    [root@localhost ~]# docker run -d -p 5000:5000 registry

    #可以访问私有仓库的镜像数据
    [root@localhost ~]# curl localhost:5000/v2/_catalog
    {"repositories":[]}

    # 将镜像daocloud.io/library/nginx标记为localhost:5000/nginx镜像
    [root@localhost ~]# docker image tag daocloud.io/library/nginx localhost:5000/nginx

    # Push 镜像
    [root@localhost ~]# docker push localhost:5000/nginx

    #可以访问私有仓库的镜像数据
    [root@localhost ~]# curl localhost:5000/v2/_catalog
    {"repositories":["nginx"]}

    # 删除标签localhost:5000/nginx:latest
    [root@localhost ~]# docker rmi localhost:5000/nginx:latest

    #查看结果，localhost:5000/nginx:latest不存在了
    [root@localhost ~]# docker images

    # Pull 镜像
    [root@localhost ~]# docker pull localhost:5000/nginx
    ```bash

参考：

https://blog.csdn.net/qq_25611295/article/details/78593182

https://www.cnblogs.com/yangleitao/p/9715790.html