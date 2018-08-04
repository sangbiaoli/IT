
1. 下载git
    ```bash
    git clone https://github.com/alibaba/dubbo.git
    ```
2. 查看远程分支
    ```bash
    cd dubbo
    $ git branch -a 
    * master
    remotes/origin/2.5.x
    remotes/origin/2.6.3-release
    remotes/origin/2.6.x
    remotes/origin/HEAD -> origin/master
    remotes/origin/master
    ```

3. 查看本地分支
    ```bash
    git branch
    * master
    ```
4. 切换分支
    ```bash
    git checkout -b dubbo-2.6.0  origin/dubbo-2.6.0 #初次切换
    git checkout master #切回主分支
    git checkout dubbo-2.6.0

    ```
5. 删除分支
    ```bash
    git branch -D master
    ```

6. 获取指定的tag
    ```bash
    git checkout -b dubbo-2.6.0 dubbo-2.6.0
    ```

7. .gitignore文件使用
    打开改文件，输入一下内容，则以后提交代码时，自动过滤这类的文件
    ```
    .idea
    target
    .settings
    *.iml
    .classpath
    .project
    ```