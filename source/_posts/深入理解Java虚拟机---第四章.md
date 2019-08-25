---
title: 深入理解Java虚拟机 - 第四章
categories: "Java"
tags:
  - Java
  - JVM
description: 深入理解Java虚拟机 - 第四章 虚拟机性能监控与故障处理工具
toc: true
date: 2018-07-18 08:30:13
---

## 第四章 虚拟机性能监控与故障处理工具
### 概述
给一个系统定位问题时，知识、经验是关键基础，数据是依据，工具是运用知识处理数据的手段。
### JDK命令行工具
#### jps: 虚拟机进程状况工具
![tools_jps_opt.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/d06485d0-c6e5-11e9-ad5a-c9a6da72bdb3.png)
![tools_jps.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/d792bfc0-c6e5-11e9-ad5a-c9a6da72bdb3.png)
- 功能：可以列出正在运行的虚拟机进程，并线上虚拟机执行的主类名称及其本地虚拟机唯一ID（LVMID）；
- 对于本地虚拟机来说，LVMID和操作系统的进程ID是一致的；
- 其他的工具通常都需要依赖jps获取LVMID；
- 主要选项：-q（只输出LVMID）、-m（输出传给main函数的参数）、-l（输出主类的全名）、-v（输出虚拟机启动JVM参数）；
#### jstat：虚拟机统计信息监视工具
![tools_jstat_opt.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/df264b30-c6e5-11e9-ad5a-c9a6da72bdb3.png)
![tools_jstat.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/0f686b70-c6e6-11e9-ad5a-c9a6da72bdb3.png)
- 功能：监视虚拟机各种运行状态信息，包括类装载、内存、垃圾收集、JIT等；
- 纯文本监控首选；
#### jinfo：Java配置信息工具
![tools_jinfo.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/19fe1350-c6e6-11e9-ad5a-c9a6da72bdb3.png)
- 功能：实时地查看虚拟机各项参数。虽然jps -v可以查看虚拟机启动参数，但是无法查看一些系统默认的参数。
- 支持运行期修改参数的能力，格式为“jinfo -flag name=value pid”；
#### jmap：Java内存映像工具
![tools_jmap_opt.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/22877070-c6e6-11e9-ad5a-c9a6da72bdb3.png)
![tools_jmap.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/2929be10-c6e6-11e9-ad5a-c9a6da72bdb3.png)
- 功能：用于生成堆转储快照（一般称为heapdump或dump文件）；
- 其他可生成heapdump的方式：使用参数-XX:+HeapDumpOnOutOfMemoryError；使用参数-XX:+HeapDumpOnCtrlBreak然后使用Ctrl+Break生成；Linux系统使用kill -3生成；
- 另外它还可以查询finalize执行队列、Java堆和永久代的详细信息；
#### jhat：虚拟机堆转储快照分析工具
![tools_jhat.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/321140c0-c6e6-11e9-ad5a-c9a6da72bdb3.png)
![tools_jhat_view.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/38238360-c6e6-11e9-ad5a-c9a6da72bdb3.png)
- 功能：用于分析jmap生成的heapdump。其内置了一个微型的HTTP服务器，可以在浏览器查看分析结果；
- 实际很少用jhat，主要有两个原因：一是分析工程会耗用服务器资源；功能相对BisualVM、IBM HeapAnalyzer较为简陋；
#### jstack：Java堆栈跟踪工具

![tools_jstack_opt.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/3dee5cc0-c6e6-11e9-ad5a-c9a6da72bdb3.png)
![tools_jstack.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/461611e0-c6e6-11e9-ad5a-c9a6da72bdb3.png)
- 功能：用于生成虚拟机当前时刻的线程快照（一般称为threaddump或javacore文件）。javacore主要目的是定位线程出现长时间停顿的原因，比如死锁、死循环、请求外部资源响应长等；
- 另外JDK 1.5后Thread类新增了getAllStackTraces()方法，可以基于此自己增加管理页面来分析；
#### HSDIS：JIT生成代码反编译
![tools_hsdis.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/4e4911a0-c6e6-11e9-ad5a-c9a6da72bdb3.png)
![tools_hsdis_result.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/543ecb90-c6e6-11e9-ad5a-c9a6da72bdb3.png)
- 现代虚拟机的实现慢慢地和虚拟机规范产生差距，如果要分析程序如果执行，最常见的就是通过软件调试工具（GDB、Windbg等）断点调试，但是对于Java来说，很多执行代码是通过JIT动态生成到CodeBuffer中的；
- 功能：HSDIS是官方推荐的HotSpot虚拟机JIT编译代码的反汇编工具，它包含在HotSpot虚拟机的源码中但没有提供编译后的程序，可以自己下载放到JDK的相关目录里；
### JDK的可视化工具
#### JConsole：Java监视与管理控制台 
- 是一种基于JMX的可视化监控和管理工具，它管理部分的功能是针对MBean进行管理，由于MBean可以使用代码、中间件服务器或者所有符合JMX规范的软件进行访问，因此这里着重介绍JConsole的监控功能；
- 通过jconsole命令启动JConsole后，会自动搜索本机所有虚拟机进程。另外还支持远程进程的监控；
- 进入主界面，支持查看以下标签页：概述、内存、线程、类、VM摘要和MBean；
#### 4.3.2 VisualVM：多合一故障处理工具
![tools_visualvm_2.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/08/25/5b8f5a90-c6e6-11e9-ad5a-c9a6da72bdb3.png)
- 目前为止JDK发布的功能最强调的运行监控和故障处理程序，另外还支持性能分析；
- VisualVM还有一个很大的优点：不需要被监视的程序基于特殊Agent运行，对应用程序的实际性能影响很小，可直接应用在生成环境中；
- VisualVM基于NetBeans平台开发，具备插件扩展功能的特性，基于插件可以做到：显示虚拟机进程以及进程配置、环境信息（jps、jinfo）、监视应用程序的CPU、GC、堆、方法区以及线程的信息（jstat、jstack）、dump以及分析堆转储快照（jmap、jhat）、方法级的程序运行性能分析，找出被调用最多运行时间最长的方法、离线程序快照（收集运行时配置、线程dump、内存dump等信息建立快照）、其他plugins的无限可能。
- 使用jvisualvm首次启动时需要在线自动安装插件（也可手工安装）；
- 特色功能：生成浏览堆转储快照（摘要、类、实例标签页、OQL控制台）、分析程序性能（Profiler页签可以录制一段时间程序每个方法执行次数和耗时）、BTrace动态日志跟踪（不停止目标程序运行的前提下通过HotSwap技术动态加入调试代码）；