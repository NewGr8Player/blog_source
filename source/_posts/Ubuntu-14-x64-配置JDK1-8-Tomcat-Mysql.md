---
title: Ubuntu 14 x64 配置JDK1.8 + Tomcat + Mysql
date: 2018-03-20 13:03:00
categories: "Linux"
tags:
     - Ubuntu
     - JDK
     - Tomcat
     - Mysql
description: 记录过程中出现的问题等
---

# 1. 首先查看系统位数[后面选择JDK位数用]
```shell
getconf LONG_BIT
```

# 2. 官网选择你想要的JDK版本、Tomcat版本[本人用的是8.5]，然后对应系统位数下载
```shell
wget [URL]
```

# 3. 下载下来解压
```shell
tar -zxvf  压缩文件名.tar.gz
#解压后在当前目录，随便移动到哪里，一会配置会用到[推荐 /usr/local/ 别的目录至少当前用户要有访问权限]
```

# 4. 配置JDK
>使用vi记事本 编辑 /etc/profile

```shell
vi /etc/profile
```

>在末尾追加内容

```shell
JAVA_HOME=/usr/local/jdk1.8
JAVA_BIN=/usr/local/jdk1.8/bin
PATH=$PATH:$JAVA_BIN
CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export JAVA_HOME JAVA_BIN PATH CLASSPATH
```
>然后保存（:wq ）,之后让修改生效

```shell
source /etc/profile
```

> 5. 测试一下Java

```shell
#查看版本
java -version
#测试编译
javac
#如果不报错，那么就说明没问题了~
```

# 6. 更改Tomcat端口

```shell
#进入Tomcat根目录
cd conf
vi server.xml
#使用命令 /8080 搜索一下 [Tomcat默认是 8080 端口]
```

# 7. 开放端口

```shell
#之前用惯了CentOS6 系列的Iptables，转到Ubuntu有点不适应，找到一个插件，先安装
apt-get install iptables-persistent
#使用 [关于iptables的规则百度谷歌有很多]
/etc/init.d/iptables-persistent {start|restart|reload|force-reload|save|flush}
#开放 80 端口 供tomcat使用 开放 3306 供Mysql使用
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
#然后
/etc/init.d/iptables-persistent save
```

# 8. 安装MySql数据库
```shell
apt-get install mysql-server-5.6 mysql-client-core-5.6
```

# 9. 新建远程用户并授权
```shell
create user username identified by password;
#授权
GRANT privileges ON databasename.tablename TO 'username'@'host' [WITH GRANT OPTION];
```

# 10. 远程无法登陆
```shell
#出现远程无法登陆问题，有两个原因：
#1. 绑定了 127.0.0.1 不允许远程访问
vi /etc/mysql/my.cnf
#注释掉里面 bin 127.0.0.1 一行
#2. 3306端口未开放
# 参考 步骤7 开放端口即可
```
