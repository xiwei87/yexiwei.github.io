---
title: Debian下LAMP的源代码配置
date: 2010-03-03 17:50:13
tags: [lamp, linux, debian]
---
#### 编译环境
```bash
Debian (Ubuntu)
apt-get install build-essential
apt-get install libncurses5-dev
sudo apt-get install libxml2-dev libcurl3-dev libpng-dev libmhash-dev libmcrypt-dev libxslt-dev libpspell-dev
```
#### Mysql编译安装参数
```bash
CHOST="i686-pc-linux-gnu" CFLAGS="-O3 -msse2 -mmmx -mfpmath=sse -mcpu=pentium4 -march=pentium4 -pipe -fomit-frame-pointer" CXXFLAGS="-O3 -msse2 -mmmx -mfpmath=sse -funroll-loops -mcpu=pentium4 -march=pentium4 -pipe -fomit-frame-pointer" ./configure –prefix=/usr/local/mysql –localstatedir=/var/lib/mysql –with-comment=Source –with-server-suffix=-Community-Server –with-mysqld-user=mysql –without-debug –with-big-tables –with-charset=utf8 –with-collation=utf8_general_ci –with-extra-charsets=all –with-pthread –enable-static –enable-thread-safe-client –with-client-ldflags=-all-static –with-mysqld-ldflags=-all-static –enable-assembler –without-ndb-debug –without-isam –with-unix-socket-path=/usr/local/mysql/tmp/mysql.sock
```
#### 配置成功会提示：
```bash
MySQL has a Web site athttp://www.mysql.com/which carries details on the
latest release, upcoming features, and other information to make your
work or play with MySQL more productive. There you can also find
information about mailing lists for MySQL discussion.

Remember to check the platform. specific part of the reference manual for
hints about installing MySQL on your platform. Also have a look at the
files in the Docs directory.

Thank you for choosing MySQL!

make
make install

groupadd mysql //增加mysql组
useradd -g mysql mysql //增加mysql用户，这个用户属于mysql组
cd /usr/local/mysql
bin/mysql_install_db –user=mysql
chown -R root:mysql . //设置权限，注意后面有一个 "."
chown -R mysql /var/lib/mysql //设置 mysql 目录权限
chgrp -R mysql . //注意后面有一个 "."
cp share/mysql/my-medium.cnf /etc/my.cnf
cp share/mysql/mysql.server /etc/init.d/mysqld //开机自动启动 mysql。
chmod 755 /etc/init.d/mysqld
rcconf //开启启动服务设置
/etc/init.d/mysqld start //启动 MySQL
bin/mysqladmin -u root password "password_for_root"
```
#### 查看mysql编译参数
```bash
cat /usr/local/mysql/bin/mysqlbug |grep ./configure
```
#### 把 mysql 加入环境变量
```bash
export PATH="$PATH:/usr/local/mysql/bin"
apache 编译
./configure //配置源代码树
–prefix=/usr/local/apache2 //体系无关文件的顶级安装目录PREFIX ，也就Apache的安装目录。
–enable-module=so //打开 so 模块，so 模块是用来提 DSO 支持的 apache 核心模块
–enable-deflate=shared //支持网页压缩
–enable-expires=shared //支持 HTTP 控制
–enable-rewrite=shared //支持 URL 重写
–enable-cache //支持缓存
–enable-file-cache //支持文件缓存
–enable-mem-cache //支持记忆缓存
–enable-disk-cache //支持磁盘缓存
–enable-static-support //支持静态连接(默认为动态连接)
–enable-static-htpasswd //使用静态连接编译 htpasswd – 管理用于基本认证的用户文件
–enable-static-htdigest //使用静态连接编译 htdigest – 管理用于摘要认证的用户文件
–enable-static-rotatelogs //使用静态连接编译 rotatelogs – 滚动 Apache 日志的管道日志程序
–enable-static-logresolve //使用静态连接编译 logresolve – 解析 Apache 日志中的IP地址为主机名
–enable-static-htdbm //使用静态连接编译 htdbm – 操作 DBM 密码数据库
–enable-static-ab //使用静态连接编译 ab – Apache HTTP 服务器性能测试工具
–enable-static-checkgid //使用静态连接编译 checkgid
–disable-cgid //禁止用一个外部 CGI 守护进程执行CGI脚本
–disable-cgi //禁止编译 CGI 版本的 PHP
–disable-userdir //禁止用户从自己的主目录中提供页面
–with-mpm=worker // 让apache以worker方式运行
–enable-authn-dbm=shared // 对动态数据库进行操作。Rewrite时需要。
make
make install
```
#### 建立一个符号连接：
```bash
ln -s /usr/local/apache2/bin/apachectl /etc/init.d/httpd
rcconf //加入自动启动
```
#### php 编译
```bash
CHOST="i686-pc-linux-gnu" CFLAGS="-O3 -msse2 -mmmx -mfpmath=sse -mcpu=pentium4 -march=pentium4 -pipe -fomit-frame-pointer" CXXFLAGS="-O3 -msse2 -mmmx -mfpmath=sse -funroll-loops -mcpu=pentium4 -march=pentium4 -pipe -fomit-frame-pointer" ./configure –prefix=/usr/local/php5 –with-mysql=/usr/local/mysql –with-gd –enable-calendar –with-zlib–with-curl –with-mysqli=/usr/local/mysql/bin/mysql_config –enable-mbstring –with-apxs2=/usr/local/apache2/bin/apxs –with-openssl –enable-zend-multibyte –with-gettext –with-mcrypt –enable-exif –with-png-dir=/usr/local/lib –enable-ftp –with-mhash –with-libxml-dir=/usr/local/lib –with-xsl –with-pspell
```
#### 配置完成提示
```bash
+——————————————————————–+
| License: |
| This software is subject to the PHP License, available in this |
| distribution in the file LICENSE. By continuing this installation |
| process, you are bound by the terms of this license agreement. |
| If you do not agree with the terms of this license, you must abort |
| the installation process at this point. |
+——————————————————————–+

Thank you for using PHP.

make
make install

cp php.ini-dist /usr/local/php/lib/php.ini
```
修改/usr/local/apache2/conf/httpd.conf，在AddType部分加入如下内容
AddType application/x-httpd-php .php

