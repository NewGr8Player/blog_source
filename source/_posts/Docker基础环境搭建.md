---
title: "Docker基础环境搭建"
date: 2018-05-17 16:14:42
categories: "Docker"
tags:
     - Linux
     - Docker
description: 
---
# 开始安装

- 由于apt官方库里的docker版本可能比较旧，所以先卸载可能存在的旧版本

```shell
sudo apt-get remove docker docker-engine docker-ce docker.io
```

- 添加`backports`源

```shell
# 编辑/etc/apt/sources.list文件,加入这句话
deb http://http.debian.net/debian jessie-backports main
# 或者
sudo add-apt-repository "deb http://http.debian.net/debian jessie-backports main"
```

- 更新`apt-get`索引

```shell
sudo apt-get update
```

- 安装Docker

```shell
sudo apt-get install docker.io
```

- 测试Docker安装结果

**查看镜像列表**

```shell
sudo docker images ls
```

# 使用非root用户运行Docker

- 将当前用户加入`docker`用户组

**指定其他用户时只需要将`${USER}`改为对应用户名即可**

```shell
sudo gpasswd -a ${USER} docker
```

- 重启Docker(命令在基于Debian的发布版上大都可用)

```shell
sudo systemctl restart docker
```

- 当前用户注销后再重新登录

```shell
# 注销
logout
```