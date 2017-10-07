---
title: 全球可信并且唯一免费的HTTPS(SSL)证书颁发机构StartSSL
date: 2010-03-30 17:42:00
tags: [ssl,free,bug]
---
HTTPS（全称：Hypertext Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容请看SSL。
它是一个URI scheme（抽象标识符体系），句法类同http:体系。用于安全的HTTP数据传输。https:URL表明它使用了HTTP，但HTTPS存在不同于HTTP的默认端口及一个加密/身份验证层（在HTTP与TCP之间）。这个系统的最初研发由网景公司进行，提供了身份验证与加密通讯方法，现在它被广泛用于万维网上安全敏感的通讯，例如交易支付方面。
#### 1、自行颁发不受浏览器信任的SSL证书：
HTTPS的SSL证书可以自行颁发，Linux下的颁发步骤如下：
```bash
openssl genrsa -des3 -out api.bz.key 1024
openssl req -new -key api.bz.key -out api.bz.csr
openssl rsa -in api.bz.key -out api.bz_nopass.key
```
![](http://zyan.cc/attachment/200911/1258146742_2397f7b9.png)
<!--more--->
Nginx.conf的SSL证书配置，使用api.bz_nopass.key，在启动Nginx是无需输入SSL证书密码，而使用api.bz.key则需要输入密码：
引用
```bash
server
{
server_name sms.api.bz;
listen  443;
index index.html index.htm index.php;
root  /data0/htdocs/api.bz;
ssl on;
ssl_certificate api.bz.crt;
ssl_certificate_key api.bz_nopass.key;
……
}
```
自行颁发的SSL证书虽然能够实现加密传输功能，但得不到浏览器的信任，会出现以下提示：
![](http://zyan.cc/attachment/200911/1258146762_2671799d.png)
#### 2、受浏览器信任的StartSSL免费SSL证书：
跟VeriSign一样，StartSSL（网址：http://www.startssl.com，公司名：StartCom）也是一家CA机构，它的根证书很久之前就被一些具有开源背景的浏览器支持（Firefox浏览器、谷歌Chrome浏览器、苹果Safari浏览器等）。
在今年9月份，StartSSL竟然搞定了微软：微软在升级补丁中，更新了通过Windows根证书认证程序（Windows Root Certificate Program）的厂商清单，并首次将StartCom公司列入了该认证清单，这是微软首次将提供免费数字验证技术的厂商加入根证书认证列表中。现在，在 Windows 7或安装了升级补丁的Windows Vista或Windows XP操作系统中，系统会完全信任由StartCom这类免费数字认证机构认证的数字证书，从而使StartSSL也得到了IE浏览器的支持。
注册成为StartSSL（http://www.startssl.com）用户，并通过邮件验证后，就可以申请免费的可信任的SSL证书了。步骤比较复杂，就不详细介绍了，申请向导的主要步骤如下： 
![](http://zyan.cc/attachment/200911/1258146831_3073b3ea.png)
![](http://zyan.cc/attachment/200911/1258146831_676782ae.png)
![](http://zyan.cc/attachment/200911/1258146831_7919f6dd.png)
#### 3、使用案例：
使用StartSSL免费SSL证书的HTTPS(SSL)网站示例：
https://sms.api.bz
![](http://zyan.cc/attachment/200911/1258147840_2397e48d.png)
![](http://zyan.cc/attachment/200911/1258150987_2337ed15.png)
#### 4、小插曲：
StartSSL虽然提供的是免费的SSL证书，但服务态度是非常不错的。之前StartSSL并不支持.bz结尾的域名，因为我有一个api.bz域名需要SSL证书，所以我发了份邮件给StartSSL的管理员，要求增加.bz域名，中间还因为.bz域名注册机构对whois查询有限制，出现了一些比较麻烦的问题，最终StartSSL通过修改程序，支持了.bz域名。
