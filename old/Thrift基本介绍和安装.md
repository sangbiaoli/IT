# thrift介绍及应用
## 一、概述
> Thrift是Apache下的一个子项目，最早是Facebook的项目，后来Facebook提供给Apache作为开源项目，在官网上，Thrift被描述为“Scalable Cross-Language Services Implementation”，说的通俗一些，Thrift具有以下特征：
1. 它有自己的跨机器的通信框架，并提供一套库。
2. 它是一个代码生成器，按照它的规则，可以生成多种编程语言的通讯过程代码。

> 一般情况下的跨机器的通信框架都是跨软件平台的（Linux,windows）, 而Thrift最特别之处在于它是跨语言的：例如，你可以用几乎所有流行语言（C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript等等）来实现通讯过程，这样做的好处就是你不用为编程语言发愁，如果服务器端与客户端都需要编写，选择你最拿手或项目规定的语言，就可以生成一个通讯框架；如果编写一个服务器端程序，定义好通讯规则（在Thrift中是.thrift文件）后，你所采用的服务器端实现语言不会影响到客户端，以后使用的人可以采用其他编程语言来实现客户端。真是件美好的事情！
        
> 与Thrift相类似的开源项目是Google的Protocol Buffer(Protobuf)，Protobuf目前提供了 C++、Java、Python 三种语言的 API，比Thrift简单一些，应用也不如Thrift广泛，有评论说Protobuf写复杂的应用比较困难。
目前thrift的版本是0.9.1，以下的讨论均以该版为基准，代码语言以C++为基准。
## 二、Thrift应用场景
> Thrift其实应分成三个部分，一个叫做Thrift代码生成器，一个叫做Thrift应用框架（库），最后一个是它生成的代码。Thrift应用的基本流程如下图所示。

![](thrift/thrift_application.png)

从上图，要生成一个Thrift应用，需用以下文件：

1. 一个.thrift文件：该文件是通信接口的定义，最主要的是信息流的格式。
2. 编程语言：这个无需解释。
3. Thrift代码生成器（Thrift compiler,翻译成代码生成器似乎更合适）：这个东西是安装thrift过程中生成的，它可以产生若干符合你约定通信格式的代码。
4. Thrift应用框架库：这个东西也是在安装过程中产生的。
5. 其他第三方支撑库：对C++来说，最主要是boost.thread、libevent，log4cxx等，按照运行的模式，生成的代码中可能需用调用这些库。


# 在CentOS 6.5安装Apache Thrift
从最低配置的安装开始，接下来的安装步骤是基于Centos 6.5上的Apache Thrift安装，该例子编译源自开发的master分支，这些说明对于高于0.9.2release版本的Apache Thrift 同样适用

## 更新系统
```bash
cd /usr/local
sudo yum -y update
```
## 安装平台开发工具
```bash
yum -y install automake libtool flex bison pkgconfig gcc-c++ boost-devel libevent-devel zlib-devel Python-devel ruby-devel crypto-utils
openssl openssl-devel
```

## 编译安装 the Apache Thrift IDL Compiler
```bash
#git clone https://git-wip-us.apache.org/repos/asf/thrift.git
git clone https://github.com/apache/thrift.git
cd thrift
git checkout -b 0.11.0 origin/0.11.0  #原来的分支过高，切换到0.11.0
./bootstrap.sh
#./configure --with-boost=/usr/include/ #因为boost安装在/usr/include里？ 
./configure --with-lua=no
make
sudo make install
```
<font color="yellow">如果make这一步出现缺少.a文件的问题：</font>
```bash
g++: error: /usr/lib64/libboost_unit_test_framework.a: No such file or directory
```
首先检查上一步的依赖项有没有全部安装，没问题的话可以看看/usr/local/lib/下有没有该文件，再拷贝到make过程中所寻找的路径下。
```bash
$ ls /usr/local/lib/libboost_unit_test_framework.*
/usr/local/lib/libboost_unit_test_framework.a
/usr/local/lib/libboost_unit_test_framework.so
/usr/local/lib/libboost_unit_test_framework.so.1.53.0

$ ls /usr/lib64/libboost_unit_test_framework*
/usr/lib64/libboost_unit_test_framework-mt.so         /usr/lib64/libboost_unit_test_framework.so
/usr/lib64/libboost_unit_test_framework-mt.so.1.53.0  /usr/lib64/libboost_unit_test_framework.so.1.53.0

$ make -n | grep libboost_unit_test_framework
/bin/sh ../../../libtool  --tag=CXX   --mode=link g++ -Wall -Wextra -pedantic -g -O2 -std=c++11 -L/usr/lib64  -o processor_test processor/ProcessorTest.o processor/EventLog.o processor/ServerThread.o libprocessortest.la ../../../lib/cpp/libthrift.la ../../../lib/cpp/libthriftnb.la /usr/lib64/libboost_unit_test_framework.a -L/usr/lib64 -levent -lrt -lpthread

$ sudo cp /usr/local/lib/libboost_unit_test_framework.a /usr/lib64/
$ make
no error.
```
<font color="yellow">make[4]: composer：命令未找到</font>
```bash
curl -sS https://getcomposer.org/installer | php  #下载composer.phar文件
mv composer.phar /usr/local/bin/composer    #移动composer.phar文件到/usr/local/bin目录下  是命令全局可用
```


<font color="yellow">Your requirements could not be resolved to an installable set of packages.</font>
```bash
  Problem 1
    - This package requires php ^5.5 || ^7.0 but your PHP version (5.4.16) does not satisfy that requirement.
  Problem 2
    - phpunit/phpunit 4.8.x-dev requires ext-dom * -> the requested PHP extension dom is missing from your system.
    - phpunit/phpunit 4.8.36 requires ext-dom * -> the requested PHP extension dom is missing from your system.
    - Installation request for phpunit/phpunit ~4.8.36 -> satisfiable by phpunit/phpunit[4.8.36, 4.8.x-dev].

  To enable extensions, verify that they are enabled in your .ini files:
    - /etc/php.ini
    - /etc/php.d/curl.ini
    - /etc/php.d/fileinfo.ini
    - /etc/php.d/json.ini
    - /etc/php.d/phar.ini
    - /etc/php.d/zip.ini
  You can also run `php --ini` inside terminal to see which files are used by PHP in CLI mode.
```

中间遇到坑，说php版本太低(centos7默认5.4),故需要升级到5.5-7.0之间，升级过程中，需要换yum，结果死活换不了，朋友帮忙换了aliyum才能更新，后续就好了。

