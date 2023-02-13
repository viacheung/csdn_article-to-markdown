---
layout:  post
title:   Linux安装MySQL8.x详细步骤
date:   22-04-11 16:25:50 修改
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-linux
-mysql
-服务器

---



一、获取mysql  [参考这篇文章](https://www.cnblogs.com/kc19941205/p/14721580.html)，但是文章中有些问题，改正了一下。 官网下载使用ftp工具上传或者使用wget指令下载 ![img](https://img-blog.csdnimg.cn/455ea85655f04ee08f0129dc69a93175.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 1.下载mysql wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz 2.解压 mysql

```java
tar xvf mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz
```

3.重命名

```java
mv mysql-8.0.20-linux-glibc2.12-x86_64 mysql-8.0
```

4.移动重命名后的文件

>为了方便管理，我们将下载后用户软件统一放到<code>usr/local</code>下

```java
mv /usr/local
```


![img](https://img-blog.csdnimg.cn/56091ff5659b42709e8942b746741c47.png) 5.切换到mysql文件夹下

```java
cd mysql-8.0
```

二、安装

6.创建data文件夹存储文件

```java
mkdir data
```


![img](https://img-blog.csdnimg.cn/9ac8e8bee7dc48158ba967af86bdf777.png) 7.创建用户组以及用户和密码

```java
groupadd mysql
useradd -g mysql mysql
```

8.授权用户

```java
chown -R mysql.mysql /usr/local/mysql-8.0
```

9.切换到bin目录下

```java
cd bin
```

10.初始化基础信息

```java
./mysqld --user=mysql --basedir=/usr/local/mysql-8.0 --datadir=/usr/local/mysql-8.0/data/ --initialize
```

如果在这里报错:

>./mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory

出现该问题首先检查该链接库文件有没有安装使用 rpm -qa|grep libaio命令进行核查 运行该命令后发现系统中无该链接库文件 使用命令，yum install libaio-devel.x86_64安装

安装成功后，继续运行数据库的初始化命令，成功后得到临时密码,记得保存，等会需要用

11.编辑my.cnf文件

```java
vi /etc/my.cnf
```

注释mysqld_safe,修改信息

```java
[mysqld]
  basedir=/usr/local/mysql-8.0/ 
  datadir=/usr/local/mysql-8.0/data/
  socket=/tmp/mysql.sock
  character-set-server=UTF8MB4
```

12.添加mysqld服务到系统 这里要切换到安装目录下执行

```java
cp -a ./support-files/mysql.server /etc/init.d/mysql
```

13.授权以及添加服务

```java
chmod +x /etc/init.d/mysql
  chkconfig --add mysql
```

14.启动mysql服务

```java
service mysql start
```

15.查看启动状态

```java
service mysql status
```

16.将mysql命令添加到服务

```java
ln -s /usr/local/mysql/mysql-8.0.20/bin/mysql /usr/bin
```

如果存在了,覆盖就执行

```java
ln -sf /usr/local/mysql/mysql-8.0.20/bin/mysql /usr/bin
```

三、登录mysql

17.用临时密码登录

```java
mysql  mysql -uroot -p
```

password填写刚才生产的临时密码，到此就说明安装成功了

18.修改root密码

```java
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```

其中123456是新的密码

19.执行 flush privileges; 使密码生效

20.选择mysql数据库 use mysql;

21.修改远程连接并生效,退出

```java
update user set host='%' where user='root';
flush privileges;
exit;
```

21.开放防火墙端口,加载生效

```java
firewall-cmd --add-port=3306/tcp --permanent
  firewall-cmd --reload
```


如果是在云服务器中安装的数据库，还需在防火墙放行MySQL端口，这样才可以远程连接。 ![img](https://img-blog.csdnimg.cn/4ff4b96cb12d4c5dab5a0ba97693b87c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

到此mysql就安装完毕

