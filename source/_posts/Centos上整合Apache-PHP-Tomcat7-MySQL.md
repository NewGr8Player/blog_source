---
title: Centos上整合Apache + PHP + Tomcat7 + MySQL
categories: "Linux"
tags:
  - Centos
  - Apache
  - PHP
  - Mysql
  - Tomcat
date: 2018-03-23 11:40:59
description: Centos上整合Apache + PHP + Tomcat7 + MySQL
---
# 安装Apahce, PHP, MySQL以及php连接mysql库的组件

```shell
yum install httpd php mysql mysql-server php-mysql
#按照需要安装，我的服务器上已经有了tomcat 和 mysql ，所有就执行如下语句
yum install httpd php php-mysql
```

# 安装apache扩展

```shell
yum install httpd-manual mod_ssl mod_perl mod_auth_mysql
```

# 安装php的常用扩展

```shell
yum install php-gd php-xml php-mbstring php-ldap php-pear php-xmlrpc
```

# 安装MySQL的扩展

```shell
yum install mysql-connector-odbc mysql-devel libdbi-dbd-mysql
```

# 配置开机启动服务

```shell
/sbin/chkconfig httpd on [设置apache httpd服务开机启动]
#按需要，我一般自己设置脚本，开机运行脚本去启动程序
```

# 安装Tomcat7

```shell
yum install java-1.6.0-openjdk java-1.6.0-openjdk-devel
wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-7/v7.0.42/bin/apache-tomcat-7.0.42.tar.gz
tar -zxvf apache-tomcat-7.0.42.tar.gz
mv apache-tomcat-7.0.42 /usr/local/tomcat7
```

# 启动Tomcat7

```shell
./usr/local/tomcat7/bin/startup.sh
```

# 测试tomcat

>在浏览器地址栏输入http://你的IP:8080/，可以看到Apache 

Tomcat的起始页，如果看不到，请确认是否是防火墙的问题。也可以修改tomcat的端口为80，此处不多说。

# Apache与Tomcat整合

```shell
#这里我们使用简单的Proxy方式整合Apache与Tomcat
vi /etc/httpd/conf.d/proxy_ajp.conf
#添加 localhost 是你本机的IP 注意不要写错，如：12.34.56.78:8009
ProxyPass /转发地址 ajp://localhost:8009/php项目名
#（已有此文件的只需将相应内容前的注释符#删除即可）
#保存修改后，重启Apache
service httpd restart
```

# 测试整合结果

```text
在浏览器地址栏输入http://你的IP/，如果看到的是Apache Tomcat的起始页，恭喜你，Apache和Tomcat的整合已经成功了！
```

# 关于无法绑定80端口

```shell
#修改conf/httpd.conf 文件内的监听ip地址为：
ServerName 0.0.0.0:80
```

# 附：以上安装的软件文件及配置的路径如下：
```text
apache的配置文件在/etc/httpd/conf下
apache的modules放在/usr/lib/httpd下
php的配置文件在/etc/php.d/下 和/etc/php.ini
php的modules放在/usr/lib/php/modules下
Tomcat7的安装目录位于/usr/local/tomcat7
```
