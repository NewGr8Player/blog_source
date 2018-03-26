---
title: Vue环境配置
date: 2018-03-26 10:25:48
categories: "VUE"
tags:
     - VUE
description: Vue环境配置
---
# 环境安装配置

## 安装NodeJs

```shell
sudo apt install nodejs-legacy
sudo apt install npm
```

## 升级npm为最新版本

```shell
sudo npm install npm@latest -g
```

**此时通过npm -v可以发现npm版本号为最新版本xxx**
## 安装用于安装nodejs的模块n

```shell
sudo npm install -g n
```

## 安装版本

```shell
#安装官方最新版本
sudo n latest
#安装官方稳定版本
sudo n stable
#安装官方最新LTS版本
sudo n lts
```

## 更换国内npm镜像

***更换后npm将变成cnpm***
## 安装webpack
```shell
sudo npm install webpack -g
```

## 安装vue脚手架

```shell
cnpm/npm install vue-cli -g
```

## 初始化项目

```shell
vue init webpack [项目名称]
```

## 运行项目

```shell
#安装项目依赖 （添加依赖或者删除时，须重新安装）
npm install
#运行开发环境
npm run dev
#运行运营环境
npm run build
```

> 运行成功可能调用浏览器或者在浏览器中输入**http://localhost:8080**访问项目。

**效果图如下**
![](http://p62t2zg97.bkt.clouddn.com/20180326101815082/VUE配置成功后浏览器截图.png)
