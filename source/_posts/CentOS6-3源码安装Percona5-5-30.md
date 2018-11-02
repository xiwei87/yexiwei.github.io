---
title: CentOS6.3源码安装Percona5.5.30
date: 2013-04-18 16:51:30
tags: [centos,mysql,percona,安装]
---
Percona 为 MySQL 数据库服务器进行了改进，在功能和性能上较 MySQL 有着很显著的提升。该版本提升了在高负载情况下的 InnoDB 的性能、为 DBA 提供一些非常有用的性能诊断工具；另外有更多的参数和命令来控制服务器行为。
Percona（mysql）从5.5版本开始，不再使用./configure编译，而是使用cmake编译器，具体的cmake编译参数可以参考mysql官网文档,(※ 非常重要)
http://dev.mysql.com/doc/refman/5.5/en/source-configuration-options.html
Percona-Server-5.5.30-rel30.2.tar.gz源码包下载地址：
http://www.percona.com/redir/downloads/Percona-Server-5.5/LATEST/source/Percona-Server-5.5.30-rel30.2.tar.gz

我的mysql目录配置如下：
安装路径：/usr/local/mysql
数据库路径：/data/mysql

准备工作：安装基本依赖包，先用yum安装cmake、automake 、autoconf ，另MySQL 5.5.x需要最少安装的包有：bison,gcc、gcc-c++、ncurses-devel
```bash
[root@localhost ~]# yum install cmake make -y
[root@localhost ~]# yum install gcc gcc-c++ autoconf bison automake zlib* fiex* libxml* ncurses-devel libmcrypt* libtool-ltdl-devel* -y
```
开始编译安装
```bash
[root@localhost ~]# tar -zxvf Percona-Server-5.5.30-rel30.2.tar.gz
[root@localhost ~]# cd Percona-Server-5.5.30-rel30.2
[root@localhost ~]# cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql
-DMYSQL_UNIX_ADDR=/data/mysql/mysql.sock
-DDEFAULT_CHARSET=utf8
-DDEFAULT_COLLATION=utf8_general_ci
-DWITH_EXTRA_CHARSETS:STRING=utf8,gbk
-DWITH_INNOBASE_STORAGE_ENGINE=1
-DWITH_READLINE=1
-DENABLED_LOCAL_INFILE=1
-DMYSQL_DATADIR=/data/mysql/
-DMYSQL_TCP_PORT=3306
[root@localhost ~]# make && make install
mysql官网英文文档简单翻译说明一下
The MyISAM, MERGE, MEMORY, and CSV engines are mandatory (always compiled into the server) and need not be installed explicitly.（说明：mysql默认支持的数据库引擎有MyISAM, MERGE, MEMORY, CSV，无需在编译时再声明）
所以上面的编译条件省掉了如下两行
-DWITH_MYISAM_STORAGE_ENGINE=1
-DWITH_MEMORY_STORAGE_ENGINE=1
但INNODB一定要声明式安装，所以多了这一行
-DWITH_INNOBASE_STORAGE_ENGINE=1
```
查看mysql.mysql的用户及组是否存在
```bash
[root@localhost ~]# cat /etc/passwd |grep mysql
mysql:x:500:500::/home/mysql:/sbin/nologin
[root@localhost ~]# cat /etc/group |grep mysql
mysql:x:500:
```
不OK就执行以下两行命令（否则跳过这一步）
```bash
[root@localhost ~]# groupadd mysql #添加mysql用户组
[root@localhost ~]# useradd mysql -g mysql -s /sbin/nologin # 
```
添加mysql用户

以下带红色字体的命令非常非常，必须要执行
```bash
[root@localhost ~]# cd /usr/local/mysql
[root@localhost ~]# chown mysql.mysql -R . #将mysql目录赋予mysql用户的执行权限
[root@localhost ~]# chown mysql.mysql -R /data/mysql
[root@localhost ~]# cp support-files/my-medium.cnf /etc/my.cnf #mysql配置文件
[root@localhost ~]# chmod 755 scripts/mysql_install_db #赋予mysql_install_db执行权限
```
以下命令为mysql 启动及自启动配置
```bash
[root@localhost ~]# scripts/mysql_install_db –user=mysql –basedir=/usr/local/mysql –datadir=/data/mysql/
[root@localhost ~]# cp support-files/mysql.server /etc/init.d/mysqld
[root@localhost ~]# chmod 755 /etc/init.d/mysqld
查看mysqld服务是否设置为开机启动
[root@localhost ~]# chkconfig –list|grep mysqld
设置为开机启动
[root@localhost ~]# chkconfig mysqld on
```
启动mysql数据库，会输出一系列有用的信息，告诉你接下去如何初始化mysql
```bash
[root@centos mysql]# service mysqld start
初始化 MySQL 数据库： Installing MySQL system tables…
OK
Filling help tables…
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/bin/mysqladmin -u root password ‘new-password’
/usr/bin/mysqladmin -u root -h centos.huoba password ‘new-password’

Alternatively you can run:
/usr/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default. This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr ; /usr/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/mysql-test ; perl mysql-test-run.pl

Please report any problems with the /usr/bin/mysqlbug script!
```
按照上述英文，我们来初始化管理员root的密码
```bash
[root@localhost ~]# /usr/local/mysql/bin/mysqladmin -u root password ‘yourpassword’
```
众所周知，mysql有两种帐号类型，即localhost和%，前者限本机连接mysql，后者可用于其它机器远程连接mysql
最后，处理帐号登录问题，让root帐号密码可以本地和远程连接使用
```bash
[root@localhost ~]# /usr/local/mysql/bin/mysql -u root -p #敲入该命令后，屏幕会提示输入密码，输入上一步设置的yourpassword
删除root密码为空的记录
mysql> use mysql;
mysql> delete from user where password=”;
mysql> flush privileges;
配置mysql允许root远程登录 #登录
mysql> grant all privileges on *.* to root@’%’ identified by “root”;
mysql> flush privileges;
mysql> quit
```
到这里Percona已经安装完毕，本文章参考了http://blog.csdn.net/yeno/article/details/8053571 （零起步6-CentOS6.3源码安装mysql5.5.28） 这边文章， 在这对原作者标识感谢！
