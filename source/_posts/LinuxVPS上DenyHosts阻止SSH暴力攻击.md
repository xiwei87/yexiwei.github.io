---
title: Linux VPS上DenyHosts阻止SSH暴力攻击
date: 2011-02-24 17:26:47
tags: [vps,linux]
---
刚装好DenyHosts，看来一下hosts.deny文件。突然吓了我一跳。
![](https://static.yexiwei.com/wp-content/uploads/2011/02/ssh.jpg)
竟然有这么IP连续登录SSH字典猜root密码，看来我装DenyHosts是明智之举。
DenyHosts它会分析/var/log/secure（redhat，Fedora Core）等日志文件，当发现同一IP在进行多次SSH密码尝试时就会记录IP到/etc/hosts.deny文件，从而达到自动屏蔽该IP的目的。
<!--more--->
DenyHosts官方网站为：http://denyhosts.sourceforge.net/
#### 1、下载DenyHosts 并解压
```bash
# wget http://soft.vpser.net/security/denyhosts/DenyHosts-2.6.tar.gz
# tar zxvf DenyHosts-2.6.tar.gz
# cd DenyHosts-2.6
```
#### 2、安装、配置和启动
```bash
# python setup.py install 默认是安装到/usr/share/denyhosts/目录的,进入相应的目录修改配置文件
# cd /usr/share/denyhosts/
# cp denyhosts.cfg-dist denyhosts.cfg
# cp daemon-control-dist daemon-control
```
默认的设置已经可以适合centos系统环境，你们可以使用vi命令查看一下denyhosts.cfg和daemon-control，里面有详细的解释
接着使用下面命令启动denyhosts程序
```bash
# chown root daemon-control
# chmod 700 daemon-control
# ./daemon-control start
```
如果要使DenyHosts每次重起后自动启动还需做如下设置：
```bash
# cd /etc/init.d
# ln -s /usr/share/denyhosts/daemon-control denyhosts
# chkconfig –add denyhosts
# chkconfig –level 2345 denyhosts on
```
或者执行下面的命令，将会修改/etc/rc.local文件：
```bash
# echo “/usr/share/denyhosts/daemon-control start” >> /etc/rc.local
```
