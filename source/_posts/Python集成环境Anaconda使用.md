---
title: Python集成环境Anaconda使用
date: 2018-03-24 16:00:28
categories: "Python"
tags:
     - Python
     - Anaconda
description: Python集成环境Anaconda使用
---
# 1. 安装
**下载地址**
[Anaconda镜像链接-清华大学开源镜像站](https://mirrors.tuna.tsinghua.edu.cn/)

# 2.windows下环境变量配置
*默认添加到系统环境变量里面的路径是**D:\Python\anaconda3**（*此处为我安装的根目录*），建议更改为如下形式，方便目录转移。*
```shell
#设置CONDA_HOME
%CONDA_HOME%
D:\Python\anaconda3
#环境中添加如下路径
%CONDA_HOME%;
%CONDA_HOME%\Scripts;
```

# 3. 多环境并存

## 3.1. 查看当前存在版本
```shell
conda info --envs
#或者
conda info -e
```
**显示如下图**
![](http://p62t2zg97.bkt.clouddn.com/Anaconda%E5%9C%A8win%E4%B8%8B%E6%95%88%E6%9E%9C.png)
***显示当前存在的python环境，带有星号的 表示是当前活动的环境。可以发现环境的名称是以envs目录下文件夹名字命名的***

## 3.2. 创建环境
通过**anaconda-navigator**或者命令新建环境。
```shell
conda create --name 环境别名 python=环境版本
```

## 3.3. 删除环境命令
```shell
conda remove --name 环境别名 --all
```
## 3.4. 切换版本
```shell
#Linux、Max OS下需要在前面加source
#激活版本
activate 版本别名
#取消激活，退回默认版本
deactivate 版本别名
```

# 4. 修改编码
python的默认编码是”ASCII“，修改为utf-8的方法：在Anaconda\Lib\site-packages目录下添加一个名字为sitecustomize.py文件，文件内容
```python
import sys  
sys.setdefaultencoding('utf-8')
```

# 5. 设置国内源
```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/  
conda config --set show_channel_urls yes 
```
# 6. 安装其他组件

## 6.1. 安装spyder
***先切换python版本然后在相应版本下操作。***
```shell
conda install spyder
```
## 6.2. 安装spyder
***先切换python版本然后在相应版本下操作。***
```shell
conda install jupyter
```

# 7. 其他常用命令
## 7.1. 包管理
```shell
# 安装 matplotlib 
conda install matplotlib
# 查看已安装的包
conda list 
# 包更新
conda update matplotlib
# 删除包
conda remove matplotlib
```
