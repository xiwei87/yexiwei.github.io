---
title: 整合现有的Nginx与MongoDB的GridFSNginx-Gridfs”的安装与配置过程
date: 2012-02-14 17:18:25
tags: [mongo,nginx]
---
环境说明：
CentOS 6 64bit Mini最小化安装
MongoDB 1.8.2
Nginx 1.0.5
本文将用费覆盖方式安装Nginx的扩展Nginx-Gridfs
Nginx的Nginx-Gridfs扩展地址：https://github.com/mdirolf/nginx-gridfs
可以使用wget下载，但是这个源码中海链接着其它源码，所有用git方法下载最方便，如果没有安装git的话按下面的步骤来
```bash
查看git包的信息
[root@vm ~]# yum info git
**这里省略了不大重要的信息**
Available Packages
Name       : git
Arch       : x86_64
Version    : 1.7.1
Release    : 2.el6_0.1
Size       : 4.6 M
Repo       : updates
Summary    : Fast Version Control System
URL        : http://git-scm.com/
License    : GPLv2
Description: Git is a fast, scalable, distributed revision control system with
           : an unusually rich command set that provides both high-level
           : operations and full access to internals.
           :
           : The git rpm installs the core tools with minimal dependencies.  To
           : install all git packages, including tools for integrating with
           : other SCMs, install the git-all meta-package.
#安装git包
[root@vm ~]# yum -y install git
```
Ok，git工具安装完成，下一步下载nginx-gridfs源码包
#### 一、安装nginx-gridfs扩展
```bash
[root@vm ~]# git clone https://github.com/mdirolf/nginx-gridfs.git
[root@vm ~]# cd nginx-gridfs
[root@vm nginx-gridfs]# git submodule init
[root@vm nginx-gridfs]# git submodule update
//进入我的nginx1.0.5的源码目录编译安装nginx-gridfs扩展
[root@vm nginx-gridfs]# cd /root/nginx-1.0.5
//编译前先查看现有的nginx的编译参数配置
[root@vm nginx-1.0.5]#/usr/local/nginx/sbin/nginx -V nginx: nginx version: nginx/1.0.5
nginx: TLS SNI support disabled
nginx: configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-ipv6 
//编译配置在原有配置基础上增加新的扩展（蓝色部分）
[root@vm nginx-1.0.5]# ./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-ipv6 <strong>--add-module=/root/nginx-gridfs</strong>
[root@vm nginx-1.0.5]# make
[root@vm nginx-1.0.5]# make install
```
Nginx的nginx-gridfs扩展模块安装完成，检查一下吧
```bash
[root@vm nginx-1.0.5]# /usr/local/nginx/sbin/nginx -V
nginx: nginx version: nginx/1.0.5
nginx: built by gcc 4.4.4 20100726 (Red Hat 4.4.4-13) (GCC)
nginx: TLS SNI support enabled
nginx: configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-ipv6 --add-module=/root/nginx-gridfs
```
##### 二、在Nginx中配置nginx-gridfs
配置语法说明：
```bash
gridfs DB_NAME [root_collection=ROOT] [field=QUERY_FIELD] [type=QUERY_TYPE] [user=USERNAME] [pass=PASSWORD]
```
* gridfs 表示告诉nginx服务器要调用gridfs模块
* root_collection= 指定Gridfs collection的前缀. 默认: fs
* field= 指定用于查询的字段 可以是 _id 和 filename. 默认: _id
* type= 指定查询的类型，这里支持 objectid, string 和int. 默认: objectid
* user= 指定数据库的用户名. 默认: NULL
* pass= 指定数据库的密码. 默认: NULL
Nginx配置文件中的具体写法：
/usr/local/nginx/conf/nginx.conf
```bash
#方法1：
location /static/ {
             gridfs ebook; #指定db 为ebook，其它均为默认，默认服务器为本地
}
方法2：
location /static/ {
        gridfs ebook
               field=filename
               type=string;
        mongo 127.0.0.1:27017;
}
方法3，用于副本集：
location /static/ {
         gridfs ebook;
                field=filename
                type=string;
         mongo "foo"
                192.168.1.60:27017
                192.168.1.61:27017;
}
方法4，这种是一个完整参数的配置例子 
location /static/ {
    gridfs ebook
           root_collection=book
           field=_id
           type=int
           user=admin
           pass=admin;
    mongo 127.0.0.1:27017;
}
```
*以上方法中 换行不是必要，仅仅是为了看的清晰,我的一个完整的配置
```bash
server
{
	listen       80;
	server_name static.ebook.vm6;

	location /
		{
			gridfs ebook
			field=filename
			type=string;
		}

	log_format  static.ebook.vm6  '$remote_addr - $remote_user [$time_local] $request '
		 '$status $body_bytes_sent $http_referer '
		 '$http_user_agent $http_x_forwarded_for';
	access_log  /home/wwwlogs/static.ebook.vm6.log  static.ebook.vm6;
}
```
注:在测试配置时要记住不要将nginx的文件过期缓存时间配置开启了，最好是在配置好服务器后再做这个工作，否则很容易造成配置错误的假象。我在配置过程用就遇到了这样的问题
到这里你也许可以正常使用Nginx-Gridfs了，其实不然。
当你重新启动系统后你会发现用浏览器访问服务器上的web站点没有响应，这是因为系统启动过程中Nginx启动比MongoDB早，它初始化的时候不能正确链接MongoDB数据库。
了解关于CentOS的守护进程启动顺序相关的解释可以看看百度百科的这篇文章：http://wenku.baidu.com/view/f13befcfa1c7aa00b52acb40.html
查看启动顺序：
```bash
[root@vm ~]# ls /etc/rc3.d
K10saslauthd   K86cgred        S02lvm2-monitor  S11auditd        S25netfs      S55sshd    S90crond
K50netconsole  K87restorecond  S08ip6tables     S12rsyslog       S26udev-post  S64mysql     S99local
K74ntpd        K89rdisc        S08iptables      S22messagebus    S50php-fpm    S80postfix
K75ntpdate     K95cgconfig     S10network       S24avahi-daemon  S55nginx      S85mongod
```
```bash
[root@vm ~]# ls /etc/rc5.d
K10saslauthd   K86cgred        S02lvm2-monitor  S11auditd        S25netfs      S55sshd    S90crond
K50netconsole  K87restorecond  S08ip6tables     S12rsyslog       S26udev-post  S64mysql     S99local
K74ntpd        K89rdisc        S08iptables      S22messagebus    S50php-fpm    S80postfix
K75ntpdate     K95cgconfig     S10network       S24avahi-daemon  S55nginx      S85mongod
```
这些都是链接到/etc/init.d/目录里的相应守护进程启动脚本
S55nginx 意思是nginx守护进程启动顺序为55
S85mongod 意思是mongod守护进程的启动顺序是85，问题就在这mongod比nginx晚启动
修改启动顺序，将mongod的启动顺序值改为比nginx小的数
```bash
[root@vm ~]# mv /etc/rc3.d/S85mongod /etc/rc3.d/S54mongod
[root@vm ~]# mv /etc/rc5.d/S85mongod /etc/rc5.d/S54mongod
```
ok，现在reboot重启下看看是不是问题已经不存在了！