---
title: Ubuntu编译WizNote
date: 2018-03-09 13:03:00
categories: "Linux"
tags:
     - Ubuntu
     - WizNote
     - QT
     - C++
description: 记录过程中出现的问题等
---

# 1. 写在最前

使用的是**Zorin OS**，基于Ubuntu的发行版。

# 2. 环境准备

## 2.1. 安装git

```shell
sudo apt-get install git
```

## 2.2. 安装编译工具

```shell
sudo apt-get install build-essential
```

## 2.3. 安装CMake

```shell
sudo apt-get install cmake
```

## 2.4. 安装ZLib

```shell
sudo apt-get install zlib1g-dev
```

## 2.5. 克隆为知笔记

```shell
cd ~
mkdir WizTeam
cd WizTeam
#这里版本号为：2.5.6
git clone https://github.com/WizTeam/WizQTClient.git
```

***项目中有Bug，产生原因是由于C++头文件引入顺序不对导致，文件路径***

```shell
src/sync/WizKMSync.cpp
```

**修正如下**

```C++
/* 把第一行的这个头文件注释掉 */
//#include "WizKMSync.h"

/* 放到这个头文件的下面 */
#include "WizKMSync_p.h"
#include "WizKMSync.h"
```

## 2.6. 安装Qt 5.7.0 for Linux 64-bit (715 MB) 或者更高版本

```shell
wget http://download.qt.io/official_releases/qt/5.7/5.7.0/qt-opensource-linux-x64-5.7.0.run 
chmod +x qt-opensource-linux-x64-5.7.0.run
./qt-opensource-linux-x64-5.7.0.run
```

# 3. 编译源代码

**运行QtCreator，菜单栏选择文件->打开文件或项目->选中CMakeLists.txt->打开**
**选中Minimum Size Release 并配置编译过后文件的保存路径**
**这部分略～**

# 4. 修复为知笔记无法切换中文输入法的问题

**经过上面的步骤编译好为知笔记的客户端之后你会发现虽然能正常登录和同步文章，但是在新建文章中切换中文输入法输入中文是无效的，下面介绍如何解决这个问题。**

## 4.1. 关于原因

> 网上给的说法是由于Qt5的兼容性问题导致为知笔记使用fcitx输入法录入汉字会有问题，解决办法是自己编译fcitx-qt5，安装部署 libfcitxplatforminputcontextplugin.so即可。

## 4.2. 安装 fcitx-libs-dev

```shell
sudo apt-get install fcitx-libs-dev
```

## 4.3. 设置qmake环境变量

> 找到你电脑上qmake的路径，我的是/home/xavier/Qt5.7.0/5.7/gcc_64/bin/

```shell
export PATH="/home/xavier/Qt5.7.0/5.7/gcc_64/bin/":$PATH
```

## 4.4. 下载fcitx-libs源码

```shell
git clone https://github.com/fcitx/fcitx-qt5.git
```

## 4.5. 编译

> 进入fcitx-qt5目录执行cmake来生成Makefile文件,使用-D参数指定Qt5_DIR路径

```shell
cd fcitx-qt5/
cmake -D Qt5_DIR=/home/eason/Qt5.7.0/5.7/gcc_64/lib/cmake/Qt5 .
# 或者直接
cmake .
make
sudo make install
```

> 将编译好的libfcitxplatforminputcontextplugin.so文件copy到/home/eason/Qt5.7.0/Tools/QtCreator/lib/Qt/plugins/platforminputcontexts目录。 重启系统。

## 4.6. 问题汇总

### 4.6.1. 问题1

```shell
xavier@newgr8player:~/fcitx-qt5$ cmake .
CMake Error at CMakeLists.txt:8 (find_package):
Could not find a package configuration file provided by "ECM" (requested
version 1.4.0) with any of the following names:
    ECMConfig.cmake
    ecm-config.cmake
Add the installation prefix of "ECM" to CMAKE_PREFIX_PATH or set "ECM_DIR"
to a directory containing one of the above files.  If "ECM" provides a
separate development package or SDK, be sure it has been installed.
-- Configuring incomplete, errors occurred!
See also "/home/xavier/fcitx-qt5/CMakeFiles/CMakeOutput.log".
xavier@newgr8player:~/fcitx-qt5$
```

***解决办法***

**先下载cmake的扩展模块包**

```shell
sudo wget https://launchpadlibrarian.net/189487929/extra-cmake-modules_1.4.0.orig.tar.xz
```

**解压**
```shell
tar -xJf extra-cmake-modules_1.4.0.orig.tar.xz
```

***进入解压后的目录,执行cmake来生成Makefile文件,使用-D参数指定Qt5_DIR路径(你的安装路径可能不一样)***

```shell
cd extra-cmake-modules-1.4.0/
cmake -D Qt5_DIR=/home/xavier/Qt5.7.0/5.7/gcc_64/lib/cmake/Qt5 .
sudo make install
```

### 4.6.2. 问题2

```shell
cmake -D Qt5_DIR=/home/xavier/Qt5.7.0/5.7/gcc_64/lib/cmake/Qt5 .
-- Could NOT find XKBCommon_XKBCommon (missing:  XKBCommon_XKBCommon_LIBRARY XKBCommon_XKBCommon_INCLUDE_DIR) 
CMake Error at /usr/share/cmake-3.5/Modules/FindPackageHandleStandardArgs.cmake:148 (message):
  Could NOT find XKBCommon (missing: XKBCommon_LIBRARIES XKBCommon) (Required
  is at least version "0.5.0")
Call Stack (most recent call first):
  /usr/share/cmake-3.5/Modules/FindPackageHandleStandardArgs.cmake:388 (_FPHSA_FAILURE_MESSAGE)
  cmake/FindXKBCommon.cmake:30 (find_package_handle_standard_args)
  CMakeLists.txt:31 (find_package)
```

**解决办法**

**下载libxkbcommon-0.5.0.tar.xz, 然后解压编译, 编译的时候需要yacc的支持,所以最先要安装bison**

```shell
sudo apt-get install bison
wget http://xkbcommon.org/download/libxkbcommon-0.5.0.tar.xz
tar -xJf libxkbcommon-0.5.0.tar.xz
cd libxkbcommon-0.5.0/
```

**使用./configure –disable-x11来生成Makefile, 这里使用–disable-x11来禁用x11,否则会报错 configure: error: xkbcommon-x11 requires xcb-xkb >= 1.10 which was not found. You can disable X11 support with –disable-x11.**


```shell
./configure --disable-x11
make
sudo make install
```
