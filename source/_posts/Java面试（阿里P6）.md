---
title: Java面试（阿里P6）
categories: Java
tags:
  - Java
  - 面试
description: 'Java面试真题-阿里P6面试'
toc: true
img: ''
author: NewGr8Player
date: 2019-09-07 08:38:54
updated: 2019-09-08 17:44:00
---

# 一面(技术面)

## ~~自我介绍和项目~~

> 这个因人而异，但自我介绍和项目多说一些简历上没有的。

## Java的内存分区

> Java程序是交由JVM执行的，所以Java内存区域划分的时候事实上是指JVM区域划分。

### 执行过程

![执行过程.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/07/a7114520-d170-11e9-a9ea-05f2a361a2f9.png)

如图所示，首先Java源代码文件(.java后缀)会被Java编译器编译为字节码文件（.class后缀），然后由JVM中的类加载器加载各个类的字节码文件，加载完毕之后，交由JVM执行引擎执行。在整个程序执行过程中，JVM会用一段空间来存储程序执行期间需要用到的数据和相关信息，这段空间一般被称作Runtime Data Area（运行时数据区）也就是我们常常说的JVM内存。因此，在Java中我们常常说到的内存管理就是针对这段空间进行管理（如何分配和回收内存空间）。

### 运行时数据区

![Runtime Data Area.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/07/6bf10ce0-d171-11e9-a9ea-05f2a361a2f9.png)

根据《Java虚拟机规范》的规定，运行时数据区通常包括这几个部分：

- Java栈（VM Stack）
- 本地方法区（Native Method Stack）
- 程序计数器（Program Counter Register）
- 方法区（Method Area）
- 堆（Heap）

如上图所示，JVM运行时数据区包括这五部分，在JVM规范中虽然规定了程序在执行期间运行时数据区应该包括这几部分，但是至于具体如何实现并没有做出规定，不同的虚拟机厂商可以有不同的实现方式。

### 运行时数据区的每部分存储了那些数据？

#### 程序计数器

`程序计数器（Program Counter Regist）`也有称作为`PC寄存器`，在汇编语言中，程序计数器是指`CPU中的寄存器`，它保存的是程序当前执行的指令地址（也可以说是下一条指令的所在存储单元地址），当CPU需要指令时，需要从程序计数器中得到当前 执行的指令所在存储单元地址，然后根据得到的地址获取到指令，在得到指令后，程序计数器便会自动加1或者根据转移指针得到下一条指令的地址，如此循环，直至执行完所有指令。
虽然JVM中的程序计数器并不像汇编语言中的程序计数器一样是物理概念上的CUP寄存器，但是JVM中的程序计数器的功能跟汇编语言中的程序计数器的功能在`逻辑上是等同的`，也就是说是用来指示`执行哪条指令的`。
由于在JVM中，多线程是通过线程轮流切换来获得CPU执行时间的，因此，在任一具体时刻，一个CPU的内核只会执行一条线程中的指令，因此，为了能够使得每个线程都在线程切换后能够恢复在切换之前的程序执行位置，每个线程都需要有自己独立的程序计数器，并且不能互相被干扰，否则就会影响到程序的正常执行次序。因此，程序计数器是每个线程所私有的。
在JVM规范中规定，如果线程执行的是`非native方法`，则程序计数器中保存的是`当前需要执行的指令的地址`；如果线程执行的是`native方法`，则程序计数器中的值是`undefined`。
由于程序计数器中存储的数据所占空间的大小不会随程序的执行而发生改变，因此，对于程序计数器不会发生`内存溢出现象(OutOfMemory)`。

#### Java栈
Java栈也称作是`虚拟机栈（Java Vitual Machine Stack）`，也就是我们常常所说的栈，跟c语言的数据段中的栈类似。事实上，Java栈是Java方法执行的内存模型。
Java栈中存放的是一个个栈帧，每个栈帧对应着一个被调用的方法，在栈帧中包括:

- 局部变量表（Local Variable）
- 操作数栈（Operaand Stack）
- 指向当前方法所属的类的运行时常量池的引用（Reference to runtime constant tool）
- 方法返回地址（Return Address）
- 一些额外的附加信息

当线程执行一个方法时，就会随之创建一个对应的栈帧，并将建立的栈帧压栈。当方法执行完毕之后，便会将栈帧出栈。因此可知，线程当前执行的方法所对应的栈帧必定位于Java栈的顶部。讲到这里，大家就应该会明白为什么 在 使用 递归方法的时候容易导致栈内存溢出的现象了以及为什么栈区的空间不用程序员去管理了（当然在Java中，程序员基本不用关系到内存分配和释放的事情，因为Java有自己的垃圾回收机制），这部分空间的分配和释放都是由系统自动实施的。对于所有的程序设计语言来说，栈这部分空间对程序员来说是不透明的。下图表示了一个Java栈的模型：

![image.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/07/b20c3a50-d177-11e9-a9ea-05f2a361a2f9.png)

- 局部变量表，顾名思义，就是用来存储放出方法中的局部变量（包括在方法中申明的非静态变量以及函数形参）。对于基本数据类型的变量，则直接存储他的值，对于应用类型的变量，存的是指向对象的应用。
- 操作栈帧，想必学过数据结构中的栈的朋友想必对表达式求值问题不会陌生，栈最典型的一个应用就是用来对表达式求值。想想一个线程执行方法的过程中，实际上就是不断执行语句的过程，而归根到底就是进行计算的过程。因此可以这么说，程序中的所有计算过程都是在借助于操作数栈来完成的。
- 指向运行时常量池的引用，因为在方法执行的过程中有可能需要用到类中的常量，所以必须要有一个引用指向运行时的常量池
- 方法返回地址，当一个方法执行完毕后，要返回之前调用它的地方，因此在栈帧中必须保存一个方法的返回地址。
- 虚拟机规范允许具体的虚拟机实现增加一些规范里没有描述的信息到栈帧中，例如与高度相关的信息，这部分信息完全取决于具体的虚拟机实现。在实际开发中，一般会把动态连接，方法返回地址与其它附加信息全部归为一类，称为栈帧信息。

> 由于每个线程执行正在执行的方法可能不同，因此每个线程都有一个Java栈，互不干扰。

#### 本地方法栈

本地方法栈与Java栈的作用和原理相似，区别只不过是Java栈是为执行Java方法服务的，而本地方法栈则是执行`本地方法（Native Method）`服务的，在JVM规范中，并没有对本地方法栈的具体实现方法以及数据结构做强制规定，虚拟机可以自由实现它，在HotSpot虚拟机中直接把本地方法栈和Java栈合二为一。

#### 堆

在c语言中，堆这部分空间是唯一一个程序员管理的内存区域，程序员可以通过`malloc函数`和`free函数`在堆上申请和释放空间。Java中的堆是用来存储对象本身以及数组（数组引用是放在Java栈中的）。只不过和c语言不通，在Java中，程序员基本不关心空间释放的问题，Java的垃圾回收机制会自动进行处理，因此这部分空间也是Java垃圾收集器管理的主要区域。另外堆是被所有线程池共享的，在JVM中只有一个堆。

#### 方法区

方法区在JVM中也是一个非常重要的区域，它与堆一样，是被线程池共享的区域。在方法区中，存储每个类的信息（包括类的名称、方法信息、字段信息）静态变量、常量以及编译器变异后的的代码等。
在class文件中除了类的字段、方法、接口等描述信息外，还有一项是常量池，用来存储编译期间生成的字面量和符号引用。
在方法区中有一个非常重要的部分就是`运行时常量池`，它是每一个类或接口的常量池的运行时表示形式，在类和接口被加载到JVM后，对应的常量池就被创建出来。当然并非class文件常量池中的内容才能进入运行时常量池，在运行期间也可将新的常量放入运行时常量池中，比如String的intern方法。

> 在JVM规范中，没有强制要求方法区必须实现垃圾回收机。


## Java对象的回收方式，回收算法

> 其实问的就是垃圾收集策略与算法
> 程序计数器、虚拟机栈、本地方法栈随线程而生，也随线程而灭；栈帧随着方法的开始而入栈，随着方法的结束而出栈。这几个区域的内存分配和回收都具有确定性，在这几个区域内不需要过多考虑回收的问题，因为方法结束或者线程结束时，内存自然就跟随着回收了。
> 而对于 Java 堆和方法区，我们只有在程序运行期间才能知道会创建哪些对象，这部分内存的分配和回收都是动态的，垃圾收集器所关注的正是这部分内存。

### 判定对象是否存活

> 若一个对象不被任何对象或变量引用，那么它就是无效对象，需要被回收。

#### 引用计数法

在对象头维护着一个 counter 计数器，对象被引用一次则计数器 +1；若引用失效则计数器 -1。当计数器为 0 时，就认为该对象无效了。

引用计数算法的实现简单，判定效率也很高，在大部分情况下它都是一个不错的算法。但是主流的 Java 虚拟机里没有选用引用计数算法来管理内存，主要是因为它很难解决对象之间循环引用的问题。

> **e.g.** `objA` 和 `objB` 都有字段`instance`表示当前对象对其他对象的引用
> objA.instance = objB
> objB.instance = objA
> 由于它们互相引用着对方，导致它们的引用计数都不为 0，于是引用计数算法无法通知 GC 收集器回收它们。

#### 可达性分析法

所有和`GC Roots`**直接**或**间接**关联的对象都是**有效对象**，和`GC Roots`没有关联的对象就是**无效对象**。

`GC Roots`是指：

- Java 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 本地方法栈中引用的对象
- 方法区中常量引用的对象
- 方法区中类静态属性引用的对象

> GC Roots 并不包括堆中对象所引用的对象，这样就不会有循环引用的问题。

### 引用的种类

> 判定对象是否存活与`引用`有关。在 JDK 1.2 以前，Java 中的引用定义很传统，一个对象只有被引用或者没有被引用两种状态，我们希望能描述这一类对象：当内存空间还足够时，则保留在内存中；如果内存空间在进行垃圾手收集后还是非常紧张，则可以抛弃这些对象。很多系统的缓存功能都符合这样的应用场景。
> 在 JDK 1.2 之后，Java 对引用的概念进行了扩充，将引用分为了以下四种。不同的引用类型，主要体现的是对象不同的可达性状态`reachable`和垃圾收集的影响。

#### 强引用（Strong Reference）

类似 "Object obj = new Object()" 这类的引用，就是强引用，只要强引用存在，垃圾收集器永远不会回收被引用的对象。但是，如果我们**错误地保持了强引用**，比如：赋值给了 static 变量，那么对象在很长一段时间内不会被回收，会产生内存泄漏。

#### 软引用（Soft Reference）

软引用是一种相对强引用弱化一些的引用，可以让对象豁免一些垃圾收集，只有当 JVM 认为内存不足时，才会去试图回收软引用指向的对象。JVM 会确保在抛出 OutOfMemoryError 之前，清理软引用指向的对象。软引用通常用来**实现内存敏感的缓存**，如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。

#### 弱引用（Weak Reference）

弱引用的**强度比软引用更弱**一些。当 JVM 进行垃圾回收时，**无论内存是否充足，都会回收**只被弱引用关联的对象。

#### 虚引用（Phantom Reference）

虚引用也称幽灵引用或者幻影引用，它是**最弱**的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响。它仅仅是提供了一种确保对象被 finalize 以后，做某些事情的机制，比如，通常用来做所谓的 Post-Mortem 清理机制。

### 回收堆中无效对象

对于可达性分析中不可达的对象，也并不是没有存活的可能。

#### 判定`finalize()`是否有必要执行

JVM 会判断此对象是否有必要执行 finalize() 方法，如果对象没有覆盖 finalize() 方法，或者 finalize() 方法已经被虚拟机调用过，那么视为“没有必要执行”。那么对象基本上就真的被回收了。

如果对象被判定为有必要执行`finalize()`方法，那么对象会被放入一个`F-Queue`队列中，虚拟机会以较低的优先级执行这些`finalize()`方法，但不会确保所有的`finalize()`方法都会执行结束。如果`finalize()`方法出现耗时操作，虚拟机就直接停止指向该方法，将对象清除。

#### 对象重生或死亡

如果在执行`finalize()`方法时，将`this`赋给了某一个引用，那么该对象就重生了。如果没有，那么就会被垃圾收集器清除。

> 任何一个对象的`finalize()`方法只会被系统`**自动**调用一次`，如果对象面临下一次回收，它的 finalize() 方法不会被再次执行，想继续在`finalize()`中自救就失效了。

### 回收方法区内存

方法区中存放生命周期较长的类信息、常量、静态变量，每次垃圾收集只有少量的垃圾被清除。方法区中主要清除两种垃圾：

- 废弃常量
- 无用的类

#### 判定废弃常量

只要常量池中的常量不被任何变量或对象引用，那么这些常量就会被清除掉。比如，一个字符串`"bingo"`进入了常量池，但是当前系统没有任何一个`String 对象`引用常量池中的`"bingo"`常量，也没有其它地方引用这个字面量，必要的话，`"bingo"常量`会被清理出常量池。

#### 判定无用的类

判定一个类是否是“无用的类”，条件较为苛刻。

- 该类的所有对象都已经被清除
- 加载该类的 ClassLoader 已经被回收
- 该类的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

> 一个类被虚拟机加载进方法区，那么在堆中就会有一个代表该类的对象：java.lang.Class。这个对象在类被加载进方法区时创建，在方法区该类被删除时清除。

### 垃圾收集算法

学会了如何判定无效对象、无用类、废弃常量之后，剩余工作就是回收这些垃圾。常见的垃圾收集算法有以下几个：

#### 标记-清除算法

**标记**的过程是：遍历所有的 `GC Roots`，然后将所有 `GC Roots` 可达的对象**标记为存活的对象**。
**清除**的过程将遍历堆中所有的对象，将没有标记的对象全部清除掉。与此同时，清除那些被标记过的对象的标记，以便下次的垃圾回收。 

这种方法有两个**不足**：

- 效率问题：标记和清除两个过程的效率都不高。
- 空间问题：标记清除之后会产生大量不连续的内存碎片，碎片太多可能导致以后需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

#### 复制算法（新生代）

为了解决效率问题，“复制”收集算法出现了。它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块内存用完，需要进行垃圾收集时，就将存活者的对象复制到另一块上面，然后将第一块内存全部清除。这种算法有优有劣：

- 优点：不会有内存碎片的问题。
- 缺点：内存缩小为原来的一半，浪费空间。

为了解决空间利用率问题，可以将内存分为三块： Eden、From Survivor、To Survivor，比例是 8:1:1，每次使用 Eden 和其中一块 Survivor。回收时，将 Eden 和 Survivor 中还存活的对象一次性复制到另外一块 Survivor 空间上，最后清理掉 Eden 和刚才使用的 Survivor 空间。这样只有 10% 的内存被浪费。

但是我们无法保证每次回收都只有不多于 10% 的对象存活，当 Survivor 空间不够，需要依赖其他内存（指老年代）进行分配担保。

##### 分配担保

为对象分配内存空间时，如果 `Eden`+`Survivor` 中空闲区域无法装下该对象，会触发 MinorGC 进行垃圾收集。但如果 Minor GC 过后依然有超过 10% 的对象存活，这样存活的对象直接通过分配担保机制进入老年代，然后再将新对象存入`Eden`区。

#### 标记-整理算法（老年代）

**标记**：它的第一个阶段与**标记/清除算法**是一模一样的，均是遍历 `GC Roots`，然后将存活的对象标记。
**整理**：移动所有**存活的对象**，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收。因此，第二阶段才称为整理阶段。

这是一种老年代的垃圾收集算法。老年代的对象一般寿命比较长，因此每次垃圾回收会有大量对象存活，如果采用复制算法，每次需要复制大量存活的对象，效率很低。

#### 分代收集算法

根据对象存活周期的不同，将内存划分为几块。一般是把 Java 堆分为新生代和老年代，针对各个年代的特点采用最适当的收集算法。  

- 新生代：复制算法
- 老年代：标记-清除算法、标记-整理算法


## CMS和G1了解么，CMS解决什么问题，说一下回收的过程。

> 这个就分别说一下CMS(Concurrent Mark-Sweep)和G1吧~

![内存区域划分示意图.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/07/d7a34400-d179-11e9-a9ea-05f2a361a2f9.png)

> `Permanent Generation`在**JDK1.8**之后被`元空间(Metaspace)`替代
> `幸存者区（Survivor Space）`的`S0`与`S1`分别叫做`From`与`To`**(没有先后对应关系，向下看)**。
> 1. 在GC开始的时候，对象只会存在于`Eden区`和`Survivor区的From`，`Survivor区的To`是空的。
> 2. 紧接着进行GC，`Eden区`中`所有存活的对象`都会被`复制`到`To`，而在`From`中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过`-XX:MaxTenuringThreshold`来设置)的对象会被移动到`老年代`中，没有达到阈值的对象会被复制到`To`。
> 
> 经过这次GC后，`Eden区`和`From`已经被清空。这个时候，`From`和`To`会交换他们的角色，也就是新的`To`就是上次GC前的`From`，新的`From`就是上次GC前的`To`。不管怎样，都会保证`Survivor区`的`To`的是空的。Minor GC会一直重复这样的过程，直到`To`被填满，`To`被填满之后，会将所有对象移动到`老年代`中。

### CMS

> `CMS(Concurrent Mark-Sweep)`是以牺牲吞吐量为代价来获得最短回收停顿时间的垃圾回收器。对于要求服务器响应速度的应用上，这种垃圾回收器非常适合。在启动JVM参数加上`-XX:+UseConcMarkSweepGC`，这个参数表示对于老年代的回收采用CMS。CMS采用的基础算法是：`标记—清除`。

#### CMS过程

- **初始标记(STW initial mark)** ：在这个阶段，需要虚拟机停顿正在执行的任务，官方的叫法STW(Stop The Word)。这个过程从垃圾回收的"根对象"开始，只扫描到能够和"根对象"直接关联的对象，并作标记。所以这个过程虽然暂停了整个JVM，但是很快就完成了。
- **并发标记(Concurrent marking)** ：这个阶段紧随初始标记阶段，在初始标记的基础上继续向下追溯标记。并发标记阶段，应用程序的线程和并发标记的线程并发执行，所以用户不会感受到停顿。
- **并发预清理(Concurrent precleaning)** ：并发预清理阶段仍然是并发的。在这个阶段，虚拟机查找在执行并发标记阶段新进入老年代的对象(可能会有一些对象从新生代晋升到老年代， 或者有一些对象被分配到老年代)。通过重新扫描，减少下一个阶段"重新标记"的工作，因为下一个阶段会Stop The World。
- **重新标记(STW remark)** ：这个阶段会暂停虚拟机，收集器线程扫描在CMS堆中剩余的对象。扫描从"跟对象"开始向下追溯，并处理对象关联。
- **并发清理(Concurrent sweeping)** ：清理垃圾对象，这个阶段收集器线程和应用程序线程并发执行。
- **并发重置(Concurrent reset)** ：这个阶段，重置CMS收集器的数据结构，等待下一次垃圾回收。

#### CMS缺点

1. CMS回收器采用的基础算法是`Mark-Sweep`。所有CMS**不会**整理、压缩堆空间。这样就会有一个问题：经过CMS收集的堆会产生`空间碎片`。 CMS不对堆空间整理压缩节约了垃圾回收的停顿时间，但也带来的堆空间的浪费。为了解决堆空间浪费问题，CMS回收器不再采用简单的指针指向一块可用堆空间来为下次对象分配使用。而是把一些未分配的空间汇总成一个列表，当JVM分配对象空间的时候，会搜索这个列表找到足够大的空间来hold住这个对象。
2. 需要更多的CPU资源。从上面的图可以看到，为了让应用程序不停顿，CMS线程和应用程序线程并发执行，这样就需要有更多的CPU，单纯靠线程切 换是不靠谱的。并且，重新标记阶段，为空保证STW快速完成，也要用到更多的甚至所有的CPU资源。当然，多核多CPU也是未来的趋势！
3. CMS的另一个缺点是它需要更大的堆空间。因为CMS标记阶段应用程序的线程还是在执行的，那么就会有堆空间继续分配的情况，为了保证在CMS回 收完堆之前还有空间分配给正在运行的应用程序，必须预留一部分空间。也就是说，CMS不会在老年代满的时候才开始收集。相反，它会尝试更早的开始收集，已避免上面提到的情况：在回收完成之前，堆没有足够空间分配！默认当老年代使用68%的时候，CMS就开始行动了。 `–XX:CMSInitiatingOccupancyFraction =n` 来设置这个阀值。

> 总得来说，CMS回收器减少了回收的停顿时间，但是降低了堆空间的利用率。

#### CMS的适用场景

> 虽然说了CMS的诸多缺点，但是他还是有自己的适用场景的，并且目前国内很多公司仍然使用者CMS算法，学习他也还是有必要的。

- 如果你的应用程序对停顿比较敏感，并且在应用程序运行的时候可以提供更大的内存和更多的CPU(也就是硬件配置较高)，那么使用CMS来收集会给你带来好处。
- 如果在JVM中，有相对较多存活时间较长的对象(老年代比较大)会更适合使用CMS。

### G1

> 在G1中，堆被划分成许多个连续的`区域(region)`。每个区域**大小相等**，在1M~32M之间。JVM最多支持2000个区域，可推算G1能支持的最大内存为2000*32M=62.5G。`区域(region)`的大小在**JVM初始化**的时候决定，也可以用`-XX:G1HeapReginSize`设置。
> 在G1中没有物理上的`Yong(Eden/Survivor)/Old Generation`，它们是逻辑的，使用一些**非连续**的区域(Region)组成的。

G1 是一款面向服务端应用的垃圾收集器，它没有新生代和老年代的概念，而是将堆划分为一块块独立的 Region。当要进行垃圾收集时，首先估计每个 Region 中垃圾的数量，每次都从垃圾回收价值最大的 Region 开始回收，因此可以获得最大的回收效率。
从整体上看， G1 是基于**标记-整理**算法实现的收集器，从局部（两个 Region 之间）上看是基于**复制**算法实现的，这意味着运行期间不会产生内存空间碎片。

> Q:一个对象和它内部所引用的对象可能不在同一个 Region 中，那么当垃圾回收时，是否需要扫描整个堆内存才能完整地进行一次可达性分析？
> A:并不！每个 Region 都有一个 Remembered Set，用于记录本区域中所有对象引用的对象所在的区域，进行可达性分析时，只要在 GC Roots 中再加上 Remembered Set 即可防止对整个堆内存进行遍历。

#### G1过程

如果不计算维护 Remembered Set 的操作，G1 收集器的工作过程分为以下几个步骤：

- 初始标记：Stop The World，仅使用一条初始标记线程对所有与 GC Roots 直接关联的对象进行标记。
- 并发标记：使用**一条**标记线程与用户线程并发执行。此过程进行可达性分析，速度很慢。
- 最终标记：Stop The World，使用多条标记线程并发执行。
- 筛选回收：回收废弃对象，此时也要 Stop The World，并使用多条筛选回收线程并发执行。

## CMS回收停顿了几次？为什么停顿？

> 首先CMS回收停顿了两次。一次是`initial mark`阶段，一次是`remark`阶段。具体请参考上一个问题CMS过程。

## Java栈什么时候会发生内存溢出，Java堆呢，说一种场景

> 这里就把常见的都说一说。

### Java栈溢出(StackOverflowError)

看下面这段代码

```Java
public class StackOverFlowTest{     
    public void stackOverFlowMethod(){    
        stackOverFlowMethod();/* 导致栈溢出的就是这句 */
    }    
    public static void main(String[] args){    
        OOMTest oom = new OOMTest();    
        oom.stackOverFlowMethod();    
    }    
} 

```

### 堆溢出(OutOfMemoryError:java heap space)

堆内存溢出的时候，虚拟机会抛出`java.lang.OutOfMemoryError:java heap space`,出现此种情况的时候，我们需要根据内存溢出的时候产生的dump文件来具体分析（需要增加`-XX:+HeapDumpOnOutOfMemoryErrorjvm`启动参数）。出现此种问题的时候有可能是内存泄露，也有可能是内存溢出了。

- 如果内存泄露，我们要找出泄露的对象是怎么被GC ROOT引用起来，然后通过引用链来具体分析泄露的原因。
- 如果出现了内存溢出问题，这往往是程序本生需要的内存大于了我们给虚拟机配置的内存，这种情况下，我们可以采用调大`-Xmx`来解决这种问题。

内存溢出代码示例

```Java
public class OOMTest{    
        public static void main(String[] args){    
                List<byte[]> buffer = new ArrayList<byte[]>();    
                buffer.add(new byte[10*1024*1024]);    
        }    

} 

```

通过如下命令启动

```shell

java -verbose:gc -Xmn10M -Xms20M -Xmx20M -XX:+PrintGC OOMTest

```

控制台会有如下输出

```shell

[GC 1180K->366K(19456K), 0.0037311 secs]    
[Full GC 366K->330K(19456K), 0.0098740 secs]    
[Full GC 330K->292K(19456K), 0.0090244 secs]    
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space    
        at OOMTest.main(OOMTest.java:7)    

```

从运行结果可以看出，JVM进行了一次`Minor gc`和两次的`Major gc`，从`Major gc`的输出可以看出，gc以后`old区`使用率为134K，而字节数组为10M，加起来大于了`old generation`的空间，所以抛出了异常，如果调整`-Xms21M,-Xmx21M`,那么就不会触发gc操作也不会出现异常了。

> 从侧面验证了一个结论：当对象大于新生代剩余内存的时候，将直接放入老年代，当老年代剩余内存还是无法放下的时候，触发垃圾收集，收集后还是不能放下就会抛出内存溢出异常了。

### 持久带溢出(OutOfMemoryError: PermGen space)

我们知道Hotspot jvm通过持久带实现了Java虚拟机规范中的方法区，而运行时的常量池就是保存在方法区中的，因此持久带溢出有可能是运行时常量池溢出，也有可能是方法区中保存的class对象没有被及时回收掉或者class信息占用的内存超过了我们配置。
当持久带溢出的时候抛出`java.lang.OutOfMemoryError: PermGen space`。

- 使用一些应用服务器的热部署的时候，我们就会遇到热部署几次以后发现内存溢出了，这种情况就是因为每次热部署的后，原来的Class没有被卸载掉。
- 如果应用程序本身比较大，涉及的类库比较多，但是我们分配给持久带的内存（通过`-XX:PermSize`和`-XX:MaxPermSize`来设置）比较小的时候也可能出现此种问题。
- 一些第三方框架，比如`spring`,`hibernate`都通过字节码生成技术（比如`CGLib`）来实现一些增强的功能，这种情况可能需要更大的方法区来存储动态生成的Class文件。

我们知道Java中字符串常量是放在常量池中的，`String.intern()`这个方法运行的时候，会检查常量池中是否存和本字符串相等的对象，如果存在直接返回对常量池中对象的引用，不存在的话，先把此字符串加入常量池，然后再返回字符串的引用。那么我们就可以通过`String.intern()`方法来模拟一下运行时常量区的溢出。

代码如下：

```Java
public class OOMTest{    
        public static void main(String... args){    
                List<String> list = new ArrayList<String>();    
                while(true){    
                        list.add(UUID.randomUUID().toString().intern());    
                }    
        }        
}

```

通过如下命令启动

```shell

java -verbose:gc -Xmn5M -Xms10M -Xmx10M -XX:MaxPermSize=1M -XX:+PrintGC OOMTest

```

控制台会有如下输出

```shell

Exception in thread "main" java.lang.OutOfMemoryError: PermGen space    
        at java.lang.String.intern(Native Method)    
        at OOMTest.main(OOMTest.java:8)   

```

### OutOfMemoryError:unable to create native thread

最后我们在来看看`java.lang.OutOfMemoryError:unable to create natvie thread`这种错误。 出现这种情况的时候，一般是下面两种情况导致的：
1. 程序创建的线程数超过了操作系统的限制。对于Linux系统，我们可以通过`ulimit -u`来查看此限制。
2. 给虚拟机分配的内存过大，导致创建线程的时候需要的native内存太少。

操作系统对每个进程的内存是有限制的，我们启动Jvm,相当于启动了一个进程，假如我们一个进程占用了**4G**的内存，那么通过下面的公式计算出来的剩余内存就是建立线程栈的时候可以用的内存。
`线程栈总可用内存` = `4G` - `(-Xmx的值)` - `(-XX:MaxPermSize的值)` - `程序计数器占用的内存`
通过上面的公式我们可以看出，`-Xmx` 和 `MaxPermSize` 的值越大，那么留给线程栈可用的空间就越小，在`-Xss`参数配置的栈容量不变的情况下，可以创建的线程数也就越小。因此如果是因为这种情况导致的`unable to create native thread`,那么要么我们增大进程所占用的总内存，或者减少`-Xmx`或者`-Xss`来达到创建更多线程的目的。

### 面试者的回答

> 这一题面试者在面试的时候回答了集合类持有对象:如果对象的引用刚好被单例所持有的话，JVM就不会回收该引用。解决方案最简单只需要将单例中持有对象的变量使用`static`修饰，并在初始化时置为`null`。

## `软引用`和`弱引用`的区别

> 这个问题引申自上一个问题
> Q:集合类如何解决OOM问题
> A:用`软引用`和`弱引用`。
> 四种引用的定义参考问题`Java对象的回收方式，回收算法`下的`引用种类`

- 在应用程序中，我们使用引用类可以边面在程序执行期间将对象留在内存中。如果我们一软引用，弱引用或虚引用的方式引用对象，这样垃圾收集器就能够随意的释放对象。如果希望尽可能的减少程序在其生命周期中所占的内存大小时，这些引用类就很有好处。
- 必须指出的一个问题:要使用这些特殊的引用类，就不能保留对对象的强引用。如果保留了对对象的强引用，那么就会浪费这些类所提供的任何好处。

- 四种引用最主要的区别就是垃圾回收器回收的时机不同：
  1. **强引用**: 我们经常使用的一种引用。基本上垃圾回收器不会主动的去回收
  2. **弱引用**: 垃圾回收器会立刻回收弱引用。
  3. **软引用**: 在JVM没有出现内存不足的情况下，垃圾回收器不会去主动回收软引用
  4. **虚引用**: 虚引用引用的字符串会被垃圾回收器回收， 自己本身会被添加到关联的引用队列中去。


## Java里的锁了解哪些

> 面试者说了Lock和synchronized

### 并发关键字

#### volatile短见字

可以这样说，volatile 关键字是 Java 虚拟机提供的**轻量级的同步机制**。 

##### 功能

volatile 有 2 个主要功能：

- **可见性**。一个线程对共享变量的修改，其他线程能够立即得知这个修改。普通变量不能做到这一点，普通变量的值在线程间传递需要通过主内存来完成。
- **禁止指令重排序**。

##### 底层原理

加入 volatile 关键字时，会多出 **lock 前缀指令**， 该 lock 前缀指令相当于内存屏障，内存屏障会提供 3 个功能：

- 在执行到内存屏障这句指令时，在其前面的操作都已完成了
- 强制将**处理器行的数据**缓存写回内存
- 一个处理器的缓存回写到内存会导致其他工作内存中的缓存失效

##### 应用场景

- 状态标记

  volatile + boolean

- DCL 单例模式（Double Check Lock，双重校验锁）

  ```java
  public class Singleton {
  
      private volatile static Singleton singleton=null;
  
      private Singleton(){}
  
      public static Singleton getSingleton(){
          if(singleton==null){
              synchronized (Singleton.class){
                  if(singleton==null){
                      singleton=new Singleton();
                  }
              }
          }
          return singleton;
      }
  }

  ```


#### synchronized关键字

**线程安全问题**:

- 存在共享数据（临界资源）
- 存在多条线程共同操作这些共享数据

解决：**同步机制**。

同一时刻有且只有一个线程在操作共享数据，其他线程必须等到该线程处理完数据后再对共享数据进行操作。

同步的前提：

- 多个线程
- 多个线程使用的是同一个锁对象

同步的弊端：

当线程相当多时，因为每个线程都会去判断同步上的锁，这是很耗费资源的，无形中会降低程序的运行效率。

##### 功能

使用 synchroinzed 进行同步，可以保证原子性（保证每个时刻只有一个线程执行同步代码）和可见性（对一个变量执行 unlock 操作之前，必须把变量值同步回主内存）。

##### 使用

synchronized 修饰的对象有几种：

- 修饰一个类。

  作用范围是 synchronized 后面括号括起来的部分

  作用对象是这个类的所有对象

  ```java
  class ClassName {
     public void method() {
        synchronized(ClassName.class) {
           // todo
        }
     }
  }

  ```

- 修饰一个方法：被修饰的方法称为同步方法。

  作用范围是整个方法

  作用对象是调用这个方法的对象

  ```java
  public synchronized void method(){
     // todo
  }

  ```

- 修饰一个静态的方法。

  作用的范围是整个方法

  作用对象是这个类的所有对象

  ```java
  public synchronized static void method() {
     // todo
  }

  ```

- 修饰一个代码块：被修饰的代码块称为同步语句块

  作用范围是大括号 {} 括起来的代码块

  作用对象是调用这个代码块的对象

> **注意**：如果锁的是类对象的话，尽管new多个实例对象，但他们仍然是属于同一个类依然会被锁住，即线程之间保证同步关系。

##### 原理

```java
public class SynchronizedDemo {
    public static void main(String[] args) {
        synchronized (SynchronizedDemo.class) { //锁住类对象
        }
        method();
    }

    private synchronized static void method() { //锁住类对象
    }
}

```

![字节码分析.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/251b85c0-d18f-11e9-a9ea-05f2a361a2f9.png)

任意一个对象都拥有自己的`Monitor`，当这个对象由同步块或者同步方法调用时， 执行方法的线程必须先获取该对象的`Monitor`才能进入同步块和同步方法， 如果没有获取到`Monitor`的线程将会被阻塞在同步块和同步方法的入口处，进入到`BLOCKED`状态。

`synchronized`同步语句块的实现使用的是`monitorenter`和`monitorexit`指令。

**monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。**

使用`synchronized`进行同步，其关键就是必须要对对象的`Monitor`进行获取， 当**线程获取`Monitor`后才能继续往下执行，否则就只能等待**。 而这个获取的过程是**互斥**的，即**同一时刻只有一个线程能够获取到 Monitor**。

上面的`SynchronizedDemo`中在执行完同步代码块之后紧接着再会去执行一个静态同步方法，而这个方法锁的对象依然就这个类对象， 那么这个正在执行的线程还需要获取该锁吗？

答案是不必的，从上图中就可以看出来， 执行静态同步方法的时候就只有一条`monitorexit`指令，并没有`monitorenter`获取锁的指令。 这就是**锁的重入性**， 即在同一线程中，线程不需要再次获取同一把锁。 `synchronized`先天具有重入性。 每个对象拥有一个**计数器**，当线程获取该对象锁后，计数器就会`+1`，释放锁后就会将计数器`-1`。

`synchronized`修饰的方法并没有`monitorenter`指令和`monitorexit`指令，取得代之的是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法，JVM 通过该 `ACC_SYNCHRONIZED` 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

##### 锁优化策略

JDK 1.6 之后对`synchronized`进行优化。

锁的 4 种状态：

- 无锁
- 偏向锁
- 轻量级锁
- 重量级锁

###### 自旋锁

在很多应用上，**共享数据的锁定状态只会持续很短一段时间**。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。

在 JDK 1.6 中引入了**自适应的自旋锁**。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

###### 锁消除

锁消除是指对于**被检测出不可能存在竞争的共享数据的锁进行消除**。

锁消除主要是通过**逃逸分析来支持**，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

对于一些看起来没有加锁的代码，其实隐式的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：

```java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}

```

`String`是一个不可变的类，编译器会对`String`的拼接自动优化。在`JDK 1.5`之前，会转化为 `StringBuffer`对象的连续`append()`操作：

```java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}

```

每个`append()`方法中都有一个同步块。虚拟机观察变量`sb`，很快就会发现它的动态作用域被限制在`concatString()`方法内部。也就是说，`sb`的所有引用永远不会逃逸到`concatString()`方法之外，其他线程无法访问到它，因此可以进行消除。

###### 锁粗化

**如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗**。

上一节的示例代码中连续的`append()`方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 `append()`操作之前直至最后一个`append()`操作之后，这样只需要加锁一次就可以了。

###### 偏向锁

在大多数情况下，**锁不仅不存在多线程竞争，而且总是由同一个线程多次获得**。偏向锁的思想是偏向于第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连`CAS`操作也不再需要。

###### 轻量级锁

轻量级锁是由偏向锁升级而来，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁争用的时候，偏向锁就会升级为轻量级锁。

**对于绝大部分的锁，在整个同步周期内都是不存在竞争的**，因此也就不需要都使用互斥量进行同步，可以**先采用`CAS`操作进行同步**，如果`CAS`失败了再改用互斥量进行同步。

#### volatile 与 synchronized 比较

- volatile 是 JVM 轻量级的同步机制，所以性能比 synchronized 要好
- volatile 只能修饰变量
  synchronized 可以修饰代码块或者方法
- 多线程访问 volatile 不会出现阻塞
  synchronized 会出现阻塞
- volatile 只能保证可见性，不能保证原子性
  synchroinzed 能保证原子性，也间接保证了可见性
- volatile 修饰的变量不会被编译器优化
  synchronized 修饰的变量可以被编译器优化

### Lock 体系

#### Condition 条件对象

条件对象是**线程同步对象中的一种**，主要用来等待某种条件的发生，条件发生后，可以唤醒等待在该条件上的一个线程或者所有线程。

**条件对象要与锁一起协同工作**。通过`ReentrantLock`的`newCondtion`获取实例对象。

```Java
ReentrantLock lock = new ReentrantLock();
Condition condition = lock.newCondition();

```

注意：

- Condition 中有`await`、`signal`、`signalAll` ，分别对应 `Object` 中放入 `wait`、`notify`、`notifyAll` 方法，其实 Condition 也有上述三种方法，改变方法名称是为了避免使用上语义的混淆。

  await 和 signal / signalAll 方法就像一个开关控制着线程 A（等待方）和线程 B（通知方）。

  ![Lock-Condition演示.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/1034c3a0-d190-11e9-a9ea-05f2a361a2f9.png)

  线程 `awaitThread` 先通过 `lock.lock()` 方法获取锁成功后调用了 `condition.await()` 方法进入**等待队列**， 

  另一个线程 signalThread 通过 `lock.lock()` 方法获取锁成功后调用了 **condition.signal / signalAll， 使得线程 awaitThread 能够有机会移入到同步队列**中， 当其他线程释放 Lock 后使得线程 awaitThread 能够有机会获取 Lock， 从而使得线程 awaitThread 能够从 await 方法中退出，然后执行后续操作。 如果 awaitThread 获取 Lock 失败会直接进入到同步队列。

- 一个 Lock 可以与多个 Condition 对象绑定。

#### AQS

AQS(AbtsractQueueSynchronized) 即同步队列器。

AQS 是一个抽象类，本身并没有实现任何同步接口的，只是通过提供**同步状态的获取和释放**来供自定义的同步组件使用。

AQS 的实现依赖内部的双向队列（底层是双向链表）。

如果当前线程获取同步状态失败，则会将该线程以及等待状态等信息封装为 Node，将其**加入同步队列的尾部，同时阻塞当前线程**，当同步状态释放时，唤醒队列的头结点。

```java
private transient volatile Node head; //同步队列的头结点
private transient volatile Node tail; //同步队列的尾结点
private volatile int state; //同步状态。
// state=0,表示同步状态可用；state=1，表示同步状态已被占用

```

#### 可重入

某个线程试图获取一个已经有该线程持有的锁，那么这个请求就会成功。“重入”意味着获取的锁的操作的粒度是“线程”而不是“调用”。重入的一种实现方法是，为每个锁关联一个**计数器**（方便解锁）和一个**所有者线程**（知道是哪个线程是可重入的）。

#### 公平锁与非公平锁

公平锁是指多个线程在等待同一个锁时，**按照申请锁的顺序来依次获取锁**。

|                            公平锁                            |                           非公平锁                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| 公平锁每次获取到锁为同步队列中的第一个节点，<br>保证请求资源时间上的绝对顺序 | 非公平锁有可能刚释放锁的线程下次继续获取该锁，<br>则有可能导致其他线程永远无法获取到锁，造成“饥饿”现象。 |
|   公平锁为了保证时间上的绝对顺序，<br>需要频繁的上下文切换   | 非公平锁会**降低一定的上下文切换**，降低性能开销<br>因此，ReentrantLock 默认选择的是非公平锁 |

#### 独占锁和共享锁

独占锁模式下，每次只能有一个线程能持有锁，ReentrantLock 就是以独占方式实现的互斥锁。

共享锁，则允许多个线程同时获取锁，并发访问共享资源，如：ReadWriteLock。

很显然，独占锁是一种悲观保守的加锁策略，它避免了读/读冲突，如果某个只读线程获取锁，则其他读线程都只能等待，这种情况下就限制了不必要的并发性，因为读操作并不会影响数据的一致性。

共享锁则是一种乐观锁，它放宽了加锁策略，允许多个执行读操作的线程同时访问共享资源。

#### ReentrantLock

ReentrantLock 即可重入锁，有 3 个内部类：Sync、FairSync 和 NonfairSync。

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    //...
}

static final class FairSync extends Sync {
    //...
}

static final class NonfairSync extends Sync {
    //...
}

```

- Sync 是一个继承 AQS 的抽象类，并发控制就是通过 Sync 实现的。

  重写了 tryRelease() , 有两个子类 FiarSync 和 NonfairSync，即公平锁和非公平锁。

- 由于 Sync 重写 tryRealese()  方法，并且 FairSync 和 NonfairSync没有再次重写该方法，所以 **公平锁和非公平锁释放锁的操作是一样的**，即**唤醒等待队列中第一个被挂起的线程**。

- 公平锁和非公平锁获取锁的方式是不同的。

  公平锁获取锁时，如果一个线程已经获取了锁，其他线程都会被挂起进入等待队列，后面来的**线程等待的时间**没有等待队列中线程等待的时间长的话，那么就会放弃获取锁，直接进入等待队列；

  非公平锁获取锁的方式是一种**抢占式**的，不考虑线程等待时间，无论是哪个线程获取了锁，则其他线程就进入等待队列。

```Java
private final Sync sync;

public ReentrantLock() { //默认是非公平锁
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) { //可设置为公平锁
    sync = fair ? new FairSync() : new NonfairSync();
}

```

#### ReentrantLock 与 synchronized 的区别

- **锁的实现**：

  synchronized 是 JVM 实现的，ReentrantLock 是 JDK 实现的。 

- **性能**：

  JDK1.6 后对 synchronized 进行了很多优化，两者的性能大致相同。

- **等待可中断**：

  当持有锁的线程长期未释放锁时，正在等待的线程可选择放弃等待，改为处理其他事情。

  ReentrantLock 是等待可中断的，synchronized  则不行。

- **公平锁**：

  公平锁是指多个线程在等待同一个锁时，**按照申请锁的顺序来依次获取锁**。

  synchronized 默认是非公平锁，ReentrantLock 既可以是公平锁，又可以是非公平锁。

- **锁绑定多个条件**：

  一个 ReentrantLock 可以绑定多个 Condition 对象。

#### LockSupport

LockSupport 位于 java.util.concurrent.locks 包下。 LockSupprot 是线程的**阻塞原语**，用来**阻塞线程**和**唤醒线程**。 

每个使用 LockSupport 的线程都会与一个许可关联， 

如果该许可可用，并且可在线程中使用，则调用 park() 将会立即返回，否则可能阻塞。 

如果许可尚不可用，则可以调用 unpark 使其可用。 

但是注意**许可不可重入**，也就是说只能调用一次 park() 方法，否则会一直阻塞。

##### LockSupport 中方法

|                     方法                      |                             说明                             |
| :-------------------------------------------: | :----------------------------------------------------------: |
|                  void park()                  | 阻塞当前线程，如果调用 unpark() 方法或者当前线程被中断， 能从 park()方法中返回 |
|           void park(Object blocker)           | 功能同park()，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查 |
|          void parkNanos(long nanos)           |   阻塞当前线程，最长不超过nanos纳秒，增加了超时返回的特性    |
|  void parkNanos(Object blocker, long nanos)   | 功能同 parkNanos(long nanos)，入参增加一个 Object 对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查 |
|         void parkUntil(long deadline)         |                 阻塞当前线程，deadline 已知                  |
| void parkUntil(Object blocker, long deadline) | 功能同 parkUntil(long deadline)，入参增加一个 Object 对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查 |
|          void unpark(Thread thread)           |                  唤醒处于阻塞状态的指定线程                  |

实际上 LockSupport 阻塞和唤醒线程的功能是**依赖于 sun.misc.Unsafe**，比如 park() 方法的功能实现则是靠unsafe.park() 方法。 另外在阻塞线程这一系列方法中还有一个很有意思的现象：每个方法都会新增一个带有Object 的阻塞对象的重载方法。 那么增加了一个 Object 对象的入参会有什么不同的地方了？

- 调用 park() 方法 dump 线程：

```

"main" #1 prio=5 os_prio=0 tid=0x02cdcc00 nid=0x2b48 waiting on condition [0x00d6f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:304)
        at learn.LockSupportDemo.main(LockSupportDemo.java:7)

```

- 调用 park(Object blocker) 方法 dump 线程:

```Java
"main" #1 prio=5 os_prio=0 tid=0x0069cc00 nid=0x6c0 waiting on condition [0x00dcf000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x048c2d18> (a java.lang.String)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at learn.LockSupportDemo.main(LockSupportDemo.java:7)

```

通过分别调用这两个方法然后 dump 线程信息可以看出， 带 Object 的 park 方法相较于无参的 park 方法会增加

```java
- parking to wait for  <0x048c2d18> (a java.lang.String)

```

这种信息就类似于记录**案发现场**，有助于工程人员能够迅速发现问题解决问题。

注意：

- synchronized 使线程阻塞，线程会进入到 BLOCKED 状态
- 调用 LockSupprt 方法阻塞线程会使线程进入到 WAITING 状态

##### LockSupport 使用示例

```java
import java.util.concurrent.locks.LockSupport;

public class LockSupportExample {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "被唤醒");
        });
        Thread t2 = new Thread(() -> {
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "被唤醒");
        });
        t1.start();
        t2.start();
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        LockSupport.unpark(t1);
        LockSupport.unpark(t2);
    }
}

```

```html
Thread-0被唤醒
Thread-1被唤醒

```

t1 线程调用`LockSupport.park()`使`t1`阻塞， 当`mian`线程睡眠`3`秒结束后通过 `LockSupport.unpark(t1)`方法唤醒`t1 线程`,`t1 线程`被唤醒执行后续操作。 另外，还有一点值得关注的是，`LockSupport.unpark(t1)`可以**通过指定线程对象唤醒指定的线程**。


## 它们(Lock和synchronized)的使用方式和实现原理有什么区别呢？

> 详见上一问的回答，已经解释的很详细了；

## synchronized锁升级的过程

> 面试者的回答：偏向锁到轻量级锁再到重量级锁
> Q:它们分别是怎么实现的，解决的是哪些问题，什么时候会发生锁升级？

![锁状态示意图.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/8293b4c0-d1fd-11e9-93b8-c3a852f09de8.png)

> 在 jdk6 之后便引入了**偏向锁**和**轻量级锁**，所以总共有4种锁状态，级别由低到高依次为：**无锁状态**、**偏向锁状态**、**轻量级锁状态**、**重量级锁状态**。这几个状态会随着竞争情况逐渐升级。

> **注意**：锁可以升级但不能降级。

> 在使用`synchronized`来同步代码块的时候，经编译后，会在代码块的起始位置插入`monitorenter`指令，在结束或异常处插入`monitorexit`指令。当执行到`monitorenter`指令时，将会尝试获取对象所对应的`monitor`的所有权，即尝试获得对象的锁。而`synchronized`用的锁是存放在`Java对象头`中的。

### Java 对象头和 Monitor

#### Java 对象头

> 以 Hotspot 虚拟机为例，Hotspot 的对象头主要包括两部分数据：**Mark Word（标记字段）**、**Klass Pointer（类型指针）**。

- **Mark Word**：默认存储对象的 HashCode，分代年龄和锁标志位信息。这些信息都是与对象自身定义无关的数据，所以 Mark Word 被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。它会根据对象的状态复用自己的存储空间，也就是说在运行期间 Mark Word 里存储的数据会随着锁标志位的变化而变化。
- **Klass Point**：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

#### Monitor
- Monitor 可以理解为一个同步工具或一种同步机制，通常被描述为一个对象。每一个 Java 对象就有一把看不见的锁，称为内部锁或者 Monitor 锁。
- Monitor 是线程私有的数据结构，每一个线程都有一个可用`monitor record`列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个`monitor`关联，同时`monitor`中有一个`Owner`字段存放拥有该锁的线程的`唯一标识`，表示该锁被这个线程占用。

### 偏向锁

#### 偏向锁的获取和撤销逻辑

1. 首先获取锁对象的`Markword`，判断是否处于`可偏向状态`。`(biased_lock=1、且 ThreadId 为空)`
2. 如果是可偏向状态，则通过`CAS`操作，把当前线程的`ID`写入到`MarkWord`
    1. 如果cas成功，那么 markword 就会变成这样。表示已经获得了锁对象的偏向锁，接着执行同步代码块
    2. 如果cas失败，说明有其他线程已经获得了偏向锁， 这种情况说明当前锁存在竞争，需要撤销已获得偏向 锁的线程，并且把它持有的锁升级为轻量级锁（这个操作需要等到全局安全点，也就是没有线程在执行字节码）才能执行
3. 如果是已偏向状态，需要检查 markword 中存储的 ThreadID 是否等于当前线程的 ThreadID a) 如果相等，不需要再次获得锁，可直接执行同步代码 块 b) 如果不相等，说明当前锁偏向于其他线程，需要撤销 偏向锁并升级到轻量级锁

#### 偏向锁的撤销

偏向锁的撤销并不是把对象恢复到无锁可偏向状态（因为 偏向锁并不存在锁释放的概念），而是在获取偏向锁的过程中，发现`cas`失败也就是存在线程竞争时，直接把被偏向的锁对象升级到被加了轻量级锁的状态。

对原持有偏向锁的线程进行撤销时，原获得偏向锁的线程 有两种情况：
1. 原获得偏向锁的线程如果已经退出了临界区，也就是同 步代码块执行完了，那么这个时候会把对象头设置成无 锁状态并且争抢锁的线程可以基于 CAS 重新偏向当前线程。
2. 如果原获得偏向锁的线程的同步代码块还没执行完，处于临界区之内，这个时候会把原获得偏向锁的线程升级 为轻量级锁后继续执行同步代码块。

> 在我们的应用开发中，绝大部分情况下一定会存在`2`个以上的线程竞争，那么如果开启偏向锁，反而会提升获取锁的资源消耗。所以可以通过`jvm`参数`UseBiasedLocking`来设置开启或关闭偏向锁

### 轻量级锁

#### 轻量级锁的加锁和解锁逻辑

锁升级为轻量级锁之后，对象的 Markword 也会进行相应 的的变化。升级为轻量级锁的过程：
1. 线程在自己的栈桢中创建锁记录`LockRecord`。
2. 将锁对象的对象头中的MarkWord复制到线程的刚刚创 建的锁记录中。
3. 将锁记录中的`Owner`指针指向锁对象。
4. 将锁对象的对象头的`MarkWord`替换为指向锁记录的指针。

#### 轻量级锁的解锁
轻量级锁的锁释放逻辑其实就是获得锁的逆向逻辑，通过`CAS`操作把线程栈帧中的`LockRecord`替换回到锁对象的`MarkWord`中，如果成功表示没有竞争。如果失败，表示当前锁存在竞争，那么轻量级锁就会膨胀成为重量级锁

### 自旋锁

> 轻量级锁在加锁过程中，用到了自旋锁

所谓自旋，就是指当有另外一个线程来竞争锁时，这个线 程会在原地循环等待，而不是把该线程给阻塞，直到那个 获得锁的线程释放锁之后，这个线程就可以马上获得锁的。

> 注意，锁在原地循环的时候，是会消耗 cpu 的，就相当于在执行一个啥也没有的`for`循环。

所以，轻量级锁适用于那些同步代码块执行的很快的场景，这样，线程原地等待很短的时间就能够获得锁了。 自旋锁的使用，其实也是有一定的概率背景，在大部分同 步代码块执行的时间都是很短的。所以通过看似无异议的 循环反而能提升锁的性能。 但是自旋必须要有一定的条件控制，否则如果一个线程执行同步代码块的时间很长，那么这个线程不断的循环反而 会消耗CPU资源。默认情况下自旋的次数是`10`次，可以通过`preBlockSpin` 来修改
在 JDK1.6 之后，引入了自适应自旋锁，自适应意味着自旋 的次数不是固定不变的，而是根据前一次在同一个锁上自 旋的时间以及锁的拥有者的状态来决定。 如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并 且持有锁的线程正在运行中，那么虚拟机就会认为这次自 旋也是很有可能再次成功，进而它将允许自旋等待持续相 对更长的时间。
如果对于某个锁，自旋很少成功获得过， 那在以后尝试获取这个锁时将可能省略掉自旋过程，直接 阻塞线程，避免浪费处理器资源

### 重量级锁

当轻量级锁膨胀到重量级锁之后，意味着线程只能被挂起 阻塞来等待被唤醒了。


## Tomcat了解么，说一下类加载器结构吧。

> 以Tomcat6为例

![Tomcat类加载器.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/59faea80-d200-11e9-93b8-c3a852f09de8.png)

> Tomcat的类加载机制是**违反了双亲委托原则**的，对于一些未加载的非基础类(`Object`,`String`等)，各个web应用自己的类加载器(`WebAppClassLoader`)会优先加载，加载不到时再交给`commonClassLoader`走双亲委托。 
> 对于JVM来说：按照这个过程可以想到，如果同样在`CLASSPATH`指定的目录中和自己工作目录中存放**相同的class**，会优先加载`CLASSPATH目录`中的文件。

> Q:既然 Tomcat 不遵循双亲委派机制，那么如果我自己定义一个恶意的HashMap，会不会有风险呢？
> A:显然不会有风险，如果有，Tomcat都运行这么多年了，那群Tomcat大神能不改进吗？ tomcat不遵循双亲委派机制，只是自定义的classLoader顺序不同，但顶层还是相同的，还是要去顶层请求classloader。

## Spring中如何让A和B两个bean按顺序加载？

### 通过标识位

我们可以在业务层自己控制A，B的初始化顺序，在A中设置一个`是否初始化的`标记，B初始化前检测A是否得以初始化，如果没有则调用A的初始化方法，所谓的`check-and-act`。

代码实现如下:
Bean A：

```Java
@Service
public class A {
 
  private static volatile boolean initialized;
 
  @Autowired
  private B b;
 
  public A() {
    System.out.println("A construct");
  }
 
  @PostConstruct
  public void init() {
    initA();
  }
 
  public boolean isInitialized() {
    return initialized;
  }
 
  public void initA() {
    if (!isInitialized()) {
      System.out.println("A init");
    }
    initialized = true;
  }
}


```

Bean B:

```Java
@Service
public class B {
 
  @Autowired
  private A a;
 
 
  public B() {
    System.out.println("B construct");
  }
 
  @PostConstruct
  public void init() {
    initB();
  }
 
 
  private void initB() {
    if (!a.isInitialized()) {
      a.initA();
    }
    System.out.println("B init");
  }
}

```

运行结果
```shell
A construct
B construct
A init
B init

```

> 这种方法好处是可以做到`Lazy initialization（懒加载）`，但是如果类似逻辑很多的话代码中到处充斥着类似代码，不优雅，所以考虑是否框架本身就可以满足我们的需要。

### 使用`@DependsOn`

`@DependsOn`只是保证的被依赖的bean先于当前bean被实例化，被创建，所以如果要采用这种方式实现bean初始化顺序的控制，那么可以把初始化逻辑放在构造函数中，但是复杂耗时的逻辑仿造构造器中是不合适的，会影响系统启动速度。

### 容器加载bean之前

Spring框架中很多地方都为我们提供了扩展点，很好的体现了`开闭原则（OCP）`。其中`BeanFactoryPostProcessor`可以允许我们在容器加载任何bean之前修改应用上下文中的`BeanDefinition`（从XML配置文件或者配置类中解析得到的bean信息，用于后续实例化bean）。

示例代码如下:

```Java
@Component
public class ABeanFactoryPostProcessor implements BeanFactoryPostProcessor {
  @Override
  public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
    A.initA();
  }
}

```

> 这种方式把A中的初始化逻辑放到了加载bean之前，很适合加载系统全局配置，但是这种方式中初始化逻辑不能依赖bean的状态。

### 事件监听器的有序性

> Spring 中的`Ordered`也是一个很重要的组件，很多逻辑中都会判断对象是否实现了`Ordered`接口，如果实现了就会先进行排序操作。比如在事件发布的时候，对获取到的`ApplicationListener`会先进行排序。所以可以利用事件监听器在处理事件时的有序性，在应用上下文`refresh`完成后，分别实现A，B中对应的初始化逻辑。

示例代码如下:

Bean A:

```Java
@Component
public class ApplicationListenerA implements ApplicationListener<ApplicationContextEvent>, Ordered {
  @Override
  public void onApplicationEvent(ApplicationContextEvent event) {
    initA();
  }
 
  @Override
  public int getOrder() {
    return Ordered.HIGHEST_PRECEDENCE; // 比 ApplicationListenerB 优先级高
  }
 
  public static void initA() {
    System.out.println("A init");
  }
}
 
```

Bean B:

```Java
@Component
public class ApplicationListenerB implements ApplicationListener<ApplicationContextEvent>, Ordered{
  @Override
  public void onApplicationEvent(ApplicationContextEvent event) {
    initB();
  }
 
  @Override
  public int getOrder() {
    return Ordered.HIGHEST_PRECEDENCE -1;
  }
 
  private void initB() {
    System.out.println("B init");
  }
}

```

> 这种方式就是站在事件响应的角度，上下文加载完成后，先实现A逻辑，然后实现B逻辑。

## 海量数据去重

> Q:10亿个数去重
> A:用hash分片做
> Q:可能不均匀
> A:用`Bitmap`
> Q:数字量更多怎么办
> A:用两个bitmap
> **注**：这题面试者的回答并没有让面试官满意。我们也就这个问题做个讨论，结果做个参考。

- 如果允许使用中间件，且硬件条件允许的情况下可以使用`MapReduce+HDFS`,参考[巧用MapReduce+HDFS，海量数据去重的五大策略](https://cloud.tencent.com/developer/article/1187611)
  - 只使用HDFS和MapReduce
  - 使用HDFS和Hbase
  - 使用HDFS，MapReduce和存储控制器
  - 使用Streaming，HDFS，MapReduce
  - 结合块技术使用MapReduce
- 参考[海量数据去重之SimHash算法简介和应用](https://cloud.tencent.com/developer/article/1121843)
- 如果再在物理内存上加了限制，就需要考虑写入文件了


# 二面(技术面)

## ~~讲一下项目。~~
> 同上，多说些简历上没有的。

## 讲一下多线程把，用到哪些写一下。

### 基本线程类

> 基本线程类指的是`Thread类`，`Runnable接口`，`Callable接口`

### 高级多线程控制类

> `ThreadLocal类`,`java.util.concurrent.atomic包下实现的Automic类`,`Lock类`,`容器类(BlockingQueue,ConcurrentHashMap等)`,`ThreadPoolExecutor(线程池)`


## 线程池由哪些组件组成，有哪些线程池，分别怎么使用，以及拒绝策略有哪些。

### 线程池由哪些些组件组成

1. **线程池管理器（ThreadPoolManager）**:用于创建并管理线程池
2. **工作线程（WorkThread）**: 线程池中线程
3. **任务接口（Task）**:每个任务必须实现的接口，以供工作线程调度任务的执行。
4. **任务队列**:用于存放没有处理的任务。提供一种缓冲机制。

### 有哪些线程池以及使用场景

> 具体代码请查阅API或者Google

1. **newCachedThreadPool**：创建一个可缓存线程池程。是一种线程数量不定的线程池，并且其最大线程数为`Integer.MAX_VALUE`，这个数是很大的，一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。但是线程池中的空闲线程都有超时限制，这个超时时长是60秒，超过60秒闲置线程就会被回收。调用`execute`将重用以前构造的线程(如果线程可用)。这类线程池比较适合执行大量的耗时较少的任务，当整个线程池都处于闲置状态时，线程池中的线程都会超时被停止。
2. **newFixedThreadPool**：创建一个定长线程池。每当提交一个任务就创建一个工作线程，当线程 处于空闲状态时，它们并不会被回收，除非线程池被关闭了，如果工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列（没有大小限制）中。由于newFixedThreadPool只有核心线程并且这些核心线程不会被回收，这样它更加快速底相应外界的请求。
3. **newScheduledThreadPool**：创建一个定长线程池。它的核心线程数量是固定的，而非核心线程数是没有限制的，并且当非核心线程闲置时会被立即回收，它可安排给定延迟后运行命令或者定期地执行。这类线程池主要用于执行定时任务和具有固定周期的重复任务。
4. **newSingleThreadExecutor**：创建一个单线程化的线程池。这类线程池内部只有一个核心线程，以无界队列方式来执行该线程，这使得这些任务之间不需要处理线程同步的问题，它确保所有的任务都在同一个线程中按顺序中执行，并且可以在任意给定的时间不会有多个线程是活动的。

### 拒绝策略有哪些

> 当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize时，如果还有任务到来就会采取任务拒绝策略。

JDK自带的拒绝策略

1. `ThreadPoolExecutor.AbortPolicy`:丢弃任务并抛出RejectedExecutionException异常。
2. `ThreadPoolExecutor.DiscardPolicy`：丢弃任务，但是不抛出异常。
3. `ThreadPoolExecutor.DiscardOldestPolicy`：丢弃队列最前面的任务，然后重新提交被拒绝的任务
4. `ThreadPoolExecutor.CallerRunsPolicy`：由调用线程（提交任务的线程）处理该任务

第三方的拒绝策略

 - Dubbo
   - 输出了一条警告级别的日志，日志内容为线程池的详细设置参数，以及线程池当前的状态，还有当前拒绝任务的一些详细信息。可以说，这条日志，使用dubbo的有过生产运维经验的或多或少是见过的，这个日志简直就是日志打印的典范，其他的日志打印的典范还有spring。得益于这么详细的日志，可以很容易定位到问题所在。
   - 输出当前线程堆栈详情，这个太有用了，当你通过上面的日志信息还不能定位问题时，案发现场的dump线程上下文信息就是你发现问题的救命稻草，这个可以参考`《dubbo线程池耗尽事件-"CyclicBarrier惹的祸"》`
   - 继续抛出拒绝执行异常，使本次任务失败，这个继承了JDK默认拒绝策略的特性
 - Netty
   类似JDK 中的 CallerRunsPolicy，舍不得丢弃任务。不同的是，CallerRunsPolicy 是直接在调用者线程执行的任务。而 Netty是新建了一个线程来处理的。所以，Netty的实现相较于调用者执行策略的使用面就可以扩展到支持高效率高性能的场景了。但是也要注意一点，Netty的实现里，在创建线程时未做任何的判断约束，也就是说只要系统还有资源就会创建新的线程来处理，直到new不出新的线程了，才会抛创建线程失败的异常
 - activeMq
   最大努力执行任务型，当触发拒绝策略时，在尝试一分钟的时间重新将任务塞进任务队列，当一分钟超时还没成功时，就抛出异常
 - PinPoint
   和其他的实现都不同。他定义了一个拒绝策略链，包装了一个拒绝策略列表，当触发拒绝策略时，会将策略链中的rejectedExecution依次执行一遍。

## 什么时候多线程会发生死锁，写一个例子吧

### 什么时候多线程会发生死锁

#### 死锁产生的原因

1. 系统资源的竞争
    通常系统中拥有的不可剥夺资源，其数量不足以满足多个进程运行的需要，使得进程在 运行过程中，会因争夺资源而陷入僵局，如磁带机、打印机等。只有对不可剥夺资源的竞争 才可能产生死锁，对可剥夺资源的竞争是不会引起死锁的。
2. 进程推进顺序非法
    进程在运行过程中，请求和释放资源的顺序不当，也同样会导致死锁。例如，并发进程 P1、P2分别保持了资源R1、R2，而进程P1申请资源R2，进程P2申请资源R1时，两者都 会因为所需资源被占用而阻塞。
信号量使用不当也会造成死锁。进程间彼此相互等待对方发来的消息，结果也会使得这 些进程间无法继续向前推进。例如，进程A等待进程B发的消息，进程B又在等待进程A 发的消息，可以看出进程A和B不是因为竞争同一资源，而是在等待对方的资源导致死锁。
3. 死锁产生的必要条件
    产生死锁必须同时满足以下四个条件，只要其中任一条件不成立，死锁就不会发生。
    1. 互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
    2. 不剥夺条件：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能 由获得该资源的进程自己来释放（只能是主动释放)。
    3. 请求和保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
    4. 循环等待条件：存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被 链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, ..., pn}，其中Pi等 待的资源被P(i+1)占有（i=0, 1, ..., n-1)，Pn等待的资源被P0占有

#### 死锁代码示例

```Java
/** 
 * 一个简单的死锁类 
 * 当DeadLock类的对象flag==1时（td1），先锁定o1,睡眠500毫秒 
 * 而td1在睡眠的时候另一个flag==0的对象（td2）线程启动，先锁定o2,睡眠500毫秒 
 * td1睡眠结束后需要锁定o2才能继续执行，而此时o2已被td2锁定； 
 * td2睡眠结束后需要锁定o1才能继续执行，而此时o1已被td1锁定； 
 * td1、td2相互等待，都需要得到对方锁定的资源才能继续执行，从而死锁。 
 */  
public class DeadLock implements Runnable {  
    public int flag = 1;  
    //静态对象是类的所有对象共享的  
    private static Object o1 = new Object();
    private static Object o2 = new Object();   
    @Override  
    public void run() {  
        System.out.println("flag=" + flag);  
        if (flag == 1) {  
            synchronized (o1) {  
                try {  
                    Thread.sleep(500);  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
                synchronized (o2) {  
                    System.out.println("1");  
                }  
            }  
        }  
        if (flag == 0) {  
            synchronized (o2) {  
                try {  
                    Thread.sleep(500);  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
                synchronized (o1) {  
                    System.out.println("0");  
                }  
            }  
        }  
    }  
  
    public static void main(String[] args) {  
          
        DeadLock td1 = new DeadLock();  
        DeadLock td2 = new DeadLock();  
        td1.flag = 1;  
        td2.flag = 0;  
        //td1,td2都处于可执行状态，但JVM线程调度先执行哪个线程是不确定的。  
        //td2的run()可能在td1的run()之前运行  
        new Thread(td1).start();  
        new Thread(td2).start();  
  
    }  
}

```

## 一个字符串集合，找出pdd并且删除。

> 皮一下，面试官是不是和pdd有什么过节？

这个题可延伸部分很多，主要注意以下几点：

1. 要考虑保证线程安全
2. 使用迭代器迭代集合内元素时，不能删除集合内元素

## Redis是单线程还是多线程，Redis的分布式怎么做？

### Redis是单线程还是多线程

> Redis是单线程的，这样设计省去了上下文切换的性能与时间消耗

### Redis的分布式怎么做

> 这里的解决方案很多，但是既然使用了分布式，那么Redis也就必须使用集群，避免单点问题。
> 在使用Redis做分布式时，Redis不仅仅能充当缓存，还可以基于Redis实现分布式锁。
> 更多解决方案细节自行百度或者Google


## RPC了解么

> 面试者回答：主要是`协议栈`+`数据格式`+`序列化方式`，然后需要有`服务注册中心`管理`生产者`和`消费者`。

详细参考[RPC原理及RPC实例分析](https://my.oschina.net/hosee/blog/711632)


## TCP三次握手的过程，如果没有第三次握手有什么问题。

> 在回答这个问题前首先我们要了解**TCP连接的建立与释放（三次握手、四次挥手）**

### TCP协议相关知识

#### TCP协议简介

> TCP是面向连接的运输层协议，它提供可靠交付的、全双工的、面向字节流的点对点服务。HTTP协议便是基于TCP协议实现的。（虽然作为应用层协议，HTTP协议并没有明确要求必须使用TCP协议作为运输层协议，但是因为HTTP协议对可靠性的的要求，默认HTTP是基于TCP协议的。若是使用UDP这种不可靠的、尽最大努力交付的运输层协议来实现HTTP的话，那么TCP协议的流量控制、可靠性保障机制等等功能就必须全部放到应用层来实现）而相比网络层更进一步，运输层着眼于应用进程间的通信，而不是网络层的主机间的通讯。我们常见的端口、套接字等概念就是由此而生。（端口代表主机上的一个应用进程、而套接字则是ip地址与端口号的合体，可以在网络范围内唯一确定一个应用进程） TCP协议的可靠传输是通过滑动窗口的方法实现的；拥塞控制则有着慢开始和拥塞避免、快重传和快恢复、RED随机早期检测几种办法。

报文格式参考

![TCP报文格式.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/be0b4c10-d20d-11e9-93b8-c3a852f09de8.png)

脑图参考

![TCP报文首部.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/42988e20-d20e-11e9-93b8-c3a852f09de8.png)

 TCP报文段的首部分为固定部分和选项部分，固定部分长20byte，而选项部分长度可变。（若整个首部长度不是4byte的整数倍的话，则需要用填充位来填充）在固定首部中，与本文密切相关的是以下几项：
- **seq（序号）**：TCP连接字节流中每一个字节都会有一个编号，而本字段的值指的是本报文段所发送数据部分第一个字节的序号。
- **ack（确认号）**：表示期望收到的下一个报文段数据部分的第一个字节的编号，编号为ack-1及以前的字节已经收到。
- **SYN**：当本字段为1时，表示这是一个连接请求或者连接接受报文。
- **ACK**：仅当本字段为1时，确认号才有效。
- **FIN**：用来释放一个连接。当本字段为1时，表示此报文段的发送端数据已发送完毕，要求释放运输连接。

#### TCP的运输连接管理

> 运输连接具有三个阶段：连接建立、数据传送以及连接释放。运输连接管理就是对连接建立以及连接释放过程的管控，使得其能正常运行，达到这些目的：使通信双方能够确知对方的存在、可以允许通信双方协商一些参数（最大报文段长度、最大窗口大小等等）、能够对运输实体资源进行分配（缓存大小等）。TCP连接的建立采用客户-服务器模式：主动发起连接建立的应用进程叫做客户，被动等待连接建立的应用进程叫做服务器。

**连接建立阶段**
1. **第一次握手**：客户端的应用进程主动打开，并向客户端发出请求报文段。其首部中：`SYN=1,seq=x`。
2. **第二次握手**：服务器应用进程被动打开。若同意客户端的请求，则发回确认报文，其首部中：`SYN=1,ACK=1,ack=x+1,seq=y`。
3. **第三次握手**：客户端收到确认报文之后，通知上层应用进程连接已建立，并向服务器发出确认报文，其首部：`ACK=1,ack=y+1`。当服务器收到客户端的确认报文之后，也通知其上层应用进程连接已建立。

> 在这个过程中，通信双方的状态如下图，其中CLOSED：关闭状态、LISTEN：收听状态、SYN-SENT：同步已发送、SYN-RCVD：同步收到、ESTAB-LISHED：连接已建立。

**连接释放阶段**

1. 第一次挥手：数据传输结束以后，客户端的应用进程发出连接释放报文段，并停止发送数据，其首部：`FIN=1,seq=u`。
2. 第二次挥手：服务器端收到连接释放报文段之后，发出确认报文，其首部：`ack=u+1,seq=v`。此时本次连接就进入了半关闭状态，客户端不再向服务器发送数据。而服务器端仍会继续发送。
3. 第三次挥手：若服务器已经没有要向客户端发送的数据，其应用进程就通知服务器释放TCP连接。这个阶段服务器所发出的最后一个报文的首部应为：`FIN=1,ACK=1,seq=w,ack=u+1`。
4. 第四次挥手：客户端收到连接释放报文段之后，必须发出确认：`ACK=1,seq=u+1,ack=w+1`。 再经过`2MSL(最长报文端寿命)`后，本次TCP连接真正结束，通信双方完成了他们的告别。

> 在这个过程中，通信双方的状态如下图，其中：
> - ESTAB-LISHED：连接建立状态
> - FIN-WAIT-1：终止等待1状态
> - FIN-WAIT-2：终止等待2状态
> - CLOSE-WAIT：关闭等待状态
> - LAST-ACK：最后确认状态
> - TIME-WAIT：时间等待状态
> - CLOSED：关闭状态

完整过程如图所示：

![TCP三次握手四次挥手.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/5f705310-d20f-11e9-93b8-c3a852f09de8.png)

![TCP三次握手四次挥手.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/93f4ff00-d20f-11e9-93b8-c3a852f09de8.png)

> Q:在握手与挥手的过程中，往复的ack与seq有什么含义？
> A:这是通信双方在通信过程中的一种确认手段，确保通信双方通信的正确性。例如小时候模仿电视剧里无线电呼叫的过程：
    - `土豆土豆，我是地瓜，你能听到吗？`
    - `地瓜地瓜，我是土豆，我能听到。` 
    若客户端的报文请求号为`土豆`，则服务器端就将返回确认号`土豆+1`（标志土豆已收到），是一种通信双方的确认手段。

> Q:在结束连接的过程中，为什么在收到服务器端的连接释放报文段之后，客户端还要继续等待2MSL之后才真正关闭TCP连接呢？
> A:这里有两个原因
>    1. 需要保证服务器端收到了客户端的最后一条确认报文。假如这条报文丢失，服务器没有接收到确认报文，就会对连接释放报文进行超时重传，而此时客户端连接已关闭，无法做出响应，就造成了服务器端不停重传连接释放报文，而无法正常进入关闭状态的状况。而等待2MSL，就可以保证服务器端收到了最终确认；若服务器端没有收到，那么在2MSL之内客户端一定会收到服务器端的重传报文，此时客户端就会重传确认报文，并重置计时器。
>    2.存在一种**已失效的连接请求报文段**，需要避免这种报文端出现在本连接中，造成异常。 这种“已失效的连接请求报文段”是这么形成的：假如客户端发出了连接请求报文，然而服务器端没有收到，于是客户端进行超时重传，再一次发送了连接请求报文，并成功建立连接。然而，第一次发送的连接请求报文并没有丢失，只是在某个网络结点中发生了长时间滞留，随后，这个最初发送的报文段到达服务器端，会使得服务器端误以为客户端发出了新的请求，造成异常。

> Q:若通信双方同时请求连接或同时请求释放连接，情况如何？
> A:这种情况虽然发生的可能性极小，但是是确实存在的，TCP也特意设计了相关机制，使得在这种情况下双方仅建立一条连接。双方同时请求连接的情况下，双方同时发出请求连接报文，并进入`SYN-SENT`状态；当收到对方的请求连接报文后，会再次发送请求连接报文，确认号为对方的`SYN+1`，并进入`SYN-RCVD`状态；当收到对方第二次发出的携带确认号的请求报文之后，会进入`ESTAB-LISHED`状态。 双方同时请求释放连接也是同样的，双方同时发出连接释放报文，并进入`FIN-WAIT-1`状态；在收到对方的报文之后，发送确认报文，并进入`CLOSING`状态；在收到对方的确认报文后，进入`TIME-WAIT`状态，等待2MSL之后关闭连接。需要注意的是，这个时候虽然不用再次发送确认报文并确认对方收到，双方仍需等待2MSL之后再关闭连接，是为了防止**已失效的连接请求报文段**的影响。

### 如果没有第三次握手有什么问题

> 没有第三步，那就是服务器发送给客户端的`SYN`没有收到回应，其实这就是**SYN洪泛攻击**。
> 感兴趣可以自己百度或者Google一下，这里不做过多解释。

SYN攻击属于DDOS攻击的一种，它利用TCP协议缺陷，通过发送大量的半连接请求，耗费CPU和内存资源。
SYN攻击除了能影响主机外，还可以危害路由器、防火墙等网络系统，事实上SYN攻击并不管目标是什么系统，只要这些系统打开TCP服务就可以实施。
服务器接收到连接请求`(syn=j)`，将此信息加入未连接队列，并发送请求包给客户`(syn=k,ack=j+1)`，此时进入`SYN_RECV`状态。当服务器未收到客户端的确认包时，重发请求包，一直到超时，才将此条目从未连接队列删除。配合IP欺骗，SYN攻击能达到很好的效果，通常，客户端在短时间内伪造大量不存在的IP地址，向服务器不断地发送syn包，服务器回复确认包，并等待客户的确认，由于源地址是不存在的，服务器需要不断的重发直至超时，这些伪造的SYN包将长时间占用未连接队列，正常的SYN请求被丢弃，目标系统运行缓慢，严重者引起网络堵塞甚至系统瘫痪。

# 三面(技术面)
## ~~自我介绍~~

> 已经麻木，进入背稿阶段~

## cap了解么，分别指什么，base呢，强一致性和弱一致性有什么方法来做，2pc了解么，说一下大概过程。

### cap分别指什么

> 这是在分布式系统中的概念，内容是以下三个要素最多只能同时实现两点，不可能三者兼顾。

- Consistency（一致性）
- Availability（可用性）
- Partition tolerance（分区容错性）

### BASE理论

> BASE是`Basically Available（基本可用）`、`Soft state（软状态）`和`Eventually consistent（最终一致性）`的简写。BASE是对CAP中一致性和可用性权衡的结果，契合性思想是即使无法做到强一致性，但每个应用都可以根据自身的业务特点，采用适当的方式来使得系统达到**最终一致性**。

1. 基本可用
    基本可用是指分布式系统在出现不可预知的故障的时候，允许损失部分可用性。 
    1. 相应时间上的损失：正常情况下，一个在线搜索引擎需要在0.5秒之内返回给用户的查询结果，但由于出现故障，查询结果的响应时间增加到1-2秒。 
    2.功能上的损失：正常情况下，在一个电子商务网站上进行购物，消费者几乎能够顺利的完成每一笔订单，但是在一些节日大促购物高峰的时候（比如双十一），由于消费者的购物行为激增，为了保护购物系统的稳定性，部分消费者可能会被引导到一个降级页面。
2. 弱状态
    弱状态也称为软状态，和硬状态相对应，是指允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同的节点之间的数据副本进行数据的同步过程存在延迟。
3. 最终一致性
    最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质就是需要系统最终数据达到一致，而不需要实时保证系统数据的强一致性。在没有发生故障的前提下，数据达到一致状态的时间延迟，取决于网络延迟、系统负载和数据复制方案设计等因素。 
在时间工程实践中，最终一致性存在以下五类主要变种。 
    - 因果一致性 
    - 读己之所写 
    - 会话一致性 
    - 单调读一致性 
    - 单调写一致性 

### 强一致性和弱一致性有什么方法来做

#### 强一致性

##### 两阶段提交（2PC）

> 两阶段提交协议把分布式事务分成两个过程，一个是准备阶段，一个是提交阶段，准备阶段和提交阶段都是由事务管理器发起的，为了接下来讲解方便，我们把事务管理器称为协调者，把资管管理器称为参与者。

- **准备阶段**：协调者向参与者发起指令，参与者评估自己的状态，如果参与者评估指令可以完成，参与者会写redo或者undo日志（这也是前面提起的`Write-Ahead Log`的一种），然后锁定资源，执行操作，但是并不提交。
- **提交阶段**：如果每个参与者明确返回准备成功，也就是预留资源和执行操作成功，协调者向参与者发起提交指令，参与者提交资源变更的事务，释放锁定的资源；如果任何一个参与者明确返回准备失败，也就是预留资源或者执行操作失败，协调者向参与者发起中止指令，参与者取消已经变更的事务，执行undo日志，释放锁定的资源。

两阶段提交事务提交示意图：

![两阶段提交事务提交示意图.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/fcb973c0-d211-11e9-93b8-c3a852f09de8.png)

两阶段提交事务回滚示意图：

![两阶段提交事务回滚示意图.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/523e4e10-d212-11e9-93b8-c3a852f09de8.png)

我们看到两阶段提交协议在准备阶段锁定资源，是一个重量级的操作，并能保证强一致性，但是实现起来复杂、成本较高，不够灵活，更重要的是它有如下致命的问题：

- **阻塞**：从上面的描述来看，对于任何一次指令必须收到明确的响应，才会继续做下一步，否则处于阻塞状态，占用的资源被一直锁定，不会被释放。
- **单点故障**：如果协调者宕机，参与者没有了协调者指挥，会一直阻塞，尽管可以通过选举新的协调者替代原有协调者，但是如果之前协调者在发送一个提交指令后宕机，而提交指令仅仅被一个参与者接受，并且参与者接收后也宕机，新上任的协调者无法处理这种情况。
- **脑裂**：协调者发送提交指令，有的参与者接收到执行了事务，有的参与者没有接收到事务，就没有执行事务，多个参与者之间是不一致的。

> 上面所有的这些问题，都是需要人工干预处理，没有自动化的解决方案，因此两阶段提交协议在正常情况下能保证系统的强一致性，但是在出现异常情况下，当前处理的操作处于错误状态，需要管理员人工干预解决，因此可用性不够好，这也符合CAP协议的一致性和可用性不能兼得的原理。

##### 三阶段提交(3pc)

> 三阶段提交协议是两阶段提交协议的改进版本。它通过超时机制解决了阻塞的问题，并且把两个阶段增加为三个阶段。

- **询问阶段**：协调者询问参与者是否可以完成指令，协调者只需要回答是还是不是，而不需要做真正的操作，这个阶段超时导致中止。
- **准备阶段**：如果在询问阶段所有的参与者都返回可以执行操作，协调者向参与者发送预执行请求，然后参与者写redo和undo日志，执行操作，但是不提交操作；如果在询问阶段任何参与者返回不能执行操作的结果，则协调者向参与者发送中止请求，这里的逻辑与两阶段提交协议的的准备阶段是相似的，这个阶段超时导致成功。
- **提交阶段**：如果每个参与者在准备阶段返回准备成功，也就是预留资源和执行操作成功，协调者向参与者发起提交指令，参与者提交资源变更的事务，释放锁定的资源；如果任何一个参与者返回准备失败，也就是预留资源或者执行操作失败，协调者向参与者发起中止指令，参与者取消已经变更的事务，执行undo日志，释放锁定的资源，这里的逻辑与两阶段提交协议的提交阶段一致。

![三阶段提交示意图.png](https://newgr8player-blog.oss-cn-beijing.aliyuncs.com/hexo-client/2019/09/08/6b749ec0-d212-11e9-93b8-c3a852f09de8.png)

然而，这里与两阶段提交协议有两个主要的不同：

- 增加了一个询问阶段，询问阶段可以确保尽可能早的发现无法执行操作而需要中止的行为，但是它并不能发现所有的这种行为，只会减少这种情况的发生。
- 在准备阶段以后，协调者和参与者执行的任务中都增加了超时，一旦超时，协调者和参与者都继续提交事务，默认为成功，这也是根据概率统计上超时后默认成功的正确性最大。

> 三阶段提交协议与两阶段提交协议相比，具有如上的优点，但是一旦发生超时，系统仍然会发生不一致，只不过这种情况很少见罢了，好处就是至少不会阻塞和永远锁定资源。

##### TTC(Try-Confirm-Cancel)

上面两节讲解了两阶段提交协议和三阶段提交协议，实际上他们能解决转账和下订单和扣库存中的分布式事务的问题，但是遇到极端情况，系统会发生阻塞或者不一致的问题，需要运营或者技术人工解决。无论两阶段还是三阶段方案中都包含多个参与者、多个阶段实现一个事务，实现复杂，性能也是一个很大的问题，因此，在互联网高并发系统中，鲜有使用两阶段提交和三阶段提交协议的场景。
阿里巴巴提出了新的TCC协议，TCC协议将一个任务拆分成Try、Confirm、Cancel，正常的流程会先执行Try，如果执行没有问题，再执行Confirm，如果执行过程中出了问题，则执行操作的逆操Cancel，从正常的流程上讲，这仍然是一个两阶段的提交协议，但是，在执行出现问题的时候，有一定的自我修复能力，如果任何一个参与者出现了问题，协调者通过执行操作的逆操作来取消之前的操作，达到最终的一致状态。
可以看出，从时序上，如果遇到极端情况下TCC会有很多问题的，例如，如果在Cancel的时候一些参与者收到指令，而一些参与者没有收到指令，整个系统仍然是不一致的，这种复杂的情况，系统首先会通过补偿的方式，尝试自动修复的，如果系统无法修复，必须由人工参与解决。
从TCC的逻辑上看，可以说TCC是简化版的三阶段提交协议，解决了两阶段提交协议的阻塞问题，但是没有解决极端情况下会出现不一致和脑裂的问题。然而，TCC通过自动化补偿手段，会把需要人工处理的不一致情况降到到最少，也是一种非常有用的解决方案，根据线人，阿里在内部的一些中间件上实现了TCC模式。
我们给出一个使用TCC的实际案例，在秒杀的场景，用户发起下单请求，应用层先查询库存，确认商品库存还有余量，则锁定库存，此时订单状态为待支付，然后指引用户去支付，由于某种原因用户支付失败，或者支付超时，系统会自动将锁定的库存解锁供其他用户秒杀。

#### 弱一致性

##### 最终一致性

最终一致性是弱一致性的一种特例，保证用户最终能够读取到某操作对系统特定数据的更新。但是随着时间的迁移，不同节点上的同一份数据总是在向趋同的方向变化。也可以简单的理解为在一段时间后，节点间的数据会最终达到一致状态。对于最终一致性最好的例子就是DNS系统，由于DNS多级缓存的实现，所以修改DNS记录后不会在全球所有DNS服务节点生效，需要等待DNS服务器缓存过期后向源服务器更新新的记录才能实现。最终一致性根据更新数据后各进程访问到数据的时间和方式的不同，又可以区分为：

1. 因果一致性：如果进程A通知进程B它已更新了一个数据项，那么进程B的后续访问将返回更新后的值，且一次写入将保证取代前一次写入。与进程A无因果关系的进程C的访问遵守一般的最终一致性规则。
2. 读己之所写一致性：当进程A自己更新一个数据项之后，它总是访问到更新过的值，绝不会看到旧值。这是因果一致性模型的一个特例。
3. 会话一致性：这是上一个模型的实用版本，它把访问存储系统的进程放到会话的上下文中。只要会话还存在，系统就保证“读己之所写”一致性。如果由于某些失败情形令会话终止，就要建立新的会话，而且系统的保证不会延续到新的会话。
4. 单调读一致性：如果进程已经看到过数据对象的某个值，那么任何后续访问都不会返回在那个值之前的值。
5. 单调写一致性：系统保证来自同一个进程的写操作顺序执行。要是系统不能保证这种程度的一致性，就非常难以编程了。

同时最终一致性的不同方式可以进行组合，从服务端角度，如何尽快将更新后的数据分布到整个系统，降低达到最终一致性的时间窗口，是提高系统的可用度和用户体验非常重要的方面。对于分布式数据系统：

- N — 数据复制的份数
- W — 更新数据是需要保证写完成的节点数
- R — 读取数据的时候需要读取的节点数

1. 如果`W+R>N`：则是强一致性，写的节点和读的节点重叠。例如对于典型的一主一备同步复制的关系型数据库。`N=2,W=2,R=1`，则不管读的是主库还是备库的数据，都是一致的。
2. 如果`W+R<=N`：则是弱一致性。例如对于一主一备异步复制的关系型数据库，`N=2,W=1,R=1`，则如果读的是备库，就可能无法读取主库已经更新过的数据，所以是弱一致性。

对于分布式系统，为了保证高可用性，一般设置N>=3。不同的N,W,R组合，是在可用性和一致性之间取一个平衡，以适应不同的应用场景。

- 如果`N=W,R=1`，任何一个写节点失效，都会导致写失败，因此可用性会降低，但是由于数据分布的N个节点是同步写入的，因此可以保证强一致性。
- 如果`N=R,W=1`，只需要一个节点写入成功即可，写性能和可用性都比较高。但是读取其他节点的进程可能不能获取更新后的数据，因此是弱一致性。这种情况下，如果`W<(N+1)/2`，并且写入的节点不重叠的话，则会存在写冲突。

## 负载均衡怎么做的呢，为什么这么做？

### 负载均衡相关知识

> 软件负载解决的两个核心问题是：**选谁**、**转发**，其中最著名的是`LVS（Linux Virtual Server）`。

#### 负载均衡分类

> 最常用的是四层和七层负载均衡。

##### 二层负载均衡

负载均衡服务器对外依然提供一个VIP（虚IP），集群中不同的机器采用相同IP地址，但机器的MAC地址不一样。当负载均衡服务器接受到请求之后，通过改写报文的目标MAC地址的方式将请求转发到目标机器实现负载均衡。

##### 三层负载均衡

和二层负载均衡类似，负载均衡服务器对外依然提供一个VIP（虚IP），但集群中不同的机器采用不同的IP地址。当负载均衡服务器接受到请求之后，根据不同的负载均衡算法，通过IP将请求转发至不同的真实服务器。

##### 四层负载均衡

四层负载均衡工作在OSI模型的传输层，由于在传输层，只有TCP/UDP协议，这两种协议中除了包含源IP、目标IP以外，还包含源端口号及目的端口号。四层负载均衡服务器在接受到客户端请求后，以后通过修改数据包的地址信息（IP+端口号）将流量转发到应用服务器。

##### 七层负载均衡

七层负载均衡工作在OSI模型的应用层，应用层协议较多，常用http、radius、DNS等。七层负载就可以基于这些协议来负载。这些应用层协议中会包含很多有意义的内容。比如同一个Web服务器的负载均衡，除了根据IP加端口进行负载外，还可根据七层的URL、浏览器类别、语言来决定是否要进行负载均衡。

> 对于一般的应用来说，有了Nginx就够了。Nginx可以用于七层负载均衡。但是对于一些大的网站，一般会采用DNS+四层负载+七层负载的方式进行多层次负载均衡。

#### 常用负载均衡工具

> 硬件负载均衡性能优越，功能全面，但价格昂贵，一般适合初期或者土豪级公司长期使用。因此软件负载均衡在互联网领域大量使用。常用的软件负载均衡软件有Nginx、LVS、HaProxy等。
> Nginx/LVS/HAProxy是目前使用最广泛的三种负载均衡软件。

##### LVS

主要用来做四层负载均衡。

##### Nginx

是一个网页服务器，它能反向代理HTTP、HTTPS,、SMTP、POP3、IMAP的协议链接，以及一个负载均衡器和一个HTTP缓存。
Nginx主要用来做七层负载均衡。
**并发性能**：官方支持每秒5万并发，实际国内一般到每秒2万并发，有优化到每秒10万并发的。具体性能看应用场景。

特点：

- **模块化设计**：良好的扩展性，可以通过模块方式进行功能扩展。
- **高可靠性**：主控进程和worker是同步实现的，一个worker出现问题，会立刻启动另一个worker。
- **内存消耗低**：一万个长连接（keep-alive）,仅消耗2.5MB内存。
- **支持热部署**：不用停止服务器，实现更新配置文件，更换日志文件、更新服务器程序版本。
- **并发能力强**：官方数据每秒支持5万并发；
- **功能丰富**：优秀的反向代理功能和灵活的负载均衡策略

##### Haproxy

主要用来做七层负载均衡。

- 静态负载均衡算法包括：轮询、比率、优先权。
- 动态负载均衡算法包括：最少连接数、最快响应速度、观察方法、预测法、动态性能分配、动态服务器补充、服务质量、服务类型、规则模式。

### 为什么这么做

此部分主要注意一下几个问题：

1. 多节点包与配置的一致性
2. 文件的存放（包含上传的文件与日志文件）
3. 会话的共享与一致性（Session），如果是基于Restful的，虽然没有了Session但也需要注意某些信息的存储与验证，例如用户信息与缓存
4. 如果是异构系统，需要注意通信报文格式的保证，比如使用消息队列，传输Json格式的数据。

## 了解过集群雪崩么？

> 类似的问题还有`缓存的雪崩`，`缓存穿透`，`缓存击穿`等

**雪崩效应产生的几种场景**

 - **流量激增**：比如异常流量、用户重试导致系统负载升高；
 - **缓存刷新**：假设A为client端，B为Server端，假设A系统请求都流向B系统，请求超出了B系统的承载能力，就会造成B系统崩溃；
 - **程序有Bug**：代码循环调用的逻辑问题，资源未释放引起的内存泄漏等问题；
 - **硬件故障**：比如宕机，机房断电，光纤被挖断等。
 - **线程同步等待**：系统间经常采用同步服务调用模式，核心服务和非核心服务共用一个线程池和消息队列。如果一个核心业务线程调用非核心线程，这个非核心线程交由第三方系统完成，当第三方系统本身出现问题，导致核心线程阻塞，一直处于等待状态，而进程间的调用是有超时限制的，最终这条线程将断掉，也可能引发雪崩；

**常见解决方案**

> 使用`超时策略`和`熔断器机制`

- **针对上述雪崩情景**：有很多应对方案，但没有一个万能的模式能够应对所有场景。
- **针对流量激增**：采用自动扩缩容以应对突发流量，或在负载均衡器上安装限流模块。
- **针对缓存刷新**：参考Cache应用中的服务过载案例研究。
- **针对硬件故障**：多机房容灾，跨机房路由，异地多活等。
- **针对同步等待**：使用Hystrix做故障隔离，熔断器机制等可以解决依赖服务不可用的问题。

## MySQL的主从复制怎么做的，具体原理是什么，有什么优缺点。

### 基于Sql语句或行复制原理

- 首先，mysql主库在事务提交时会把数据库变更作为事件Events记录在二进制文件binlog中；mysql主库上的sys_binlog控制binlog日志刷新到磁盘。
- 主库推送二进制文件binlog中的事件到从库的中继日志relay log,之后从库根据中继日志重做数据库变更操作。通过逻辑复制，以此来达到数据一致。

> Mysql通过3个线程来完成主从库之间的数据复制：其中`BinLog Dump`线程跑在主库上，`I/O线程`和`SQL线程`跑在从库上。当从库启动复制（start slave）时，首先创建`I/O线程`连接主库，主库随后创建`Binlog Dump线程`读取数据库事件并发给`I/O线程`，`I/O线程`获取到数据库事件更新到从库的中继日志`Realy log`中去，之后从库上的SQL线程读取中继日志`relay log`中更新的数据库事件并应用。

### 基于GTID原理

- 主节点更新数据时，会在事务前产生GTID，一起记录到binlog日志中。
- 从节点的I/O线程将变更的bin log，写入到本地的relay log中。
- SQL线程从relay log中获取GTID，然后对比本地binlog是否有记录（所以MySQL从节点必须要开启binary log）。
- 如果有记录，说明该GTID的事务已经执行，从节点会忽略。
- 如果没有记录，从节点就会从relay log中执行该GTID的事务，并记录到bin log。
- 在解析过程中会判断是否有主键，如果没有就用二级索引，如果有就用全部扫描。

### 优缺点

1. 基于SQL语句的复制(statement-based replication, SBR)，
  - 优点：
    历史悠久，技术成熟。
    产生的binlog文件较小，比较节省空间。
    binlog中包含了所有数据库更改信息，可以据此来审核数据库的安全等情况。
    binlog可以用于实时的还原，而不仅仅用于复制。
    主从版本可以不一样，从服务器版本可以比主服务器版本高。
  - 缺点：
    不是所有的UPDATE语句都能被复制，尤其是包含不确定操作的时候。
    调用具有不确定因素的 UDF 时复制也可能出问题
    使用以下函数的语句也无法被复制：
    - LOAD_FILE()
    - UUID()
    - USER()
    - FOUND_ROWS()
    - SYSDATE() (除非启动时启用了 --sysdate-is-now 选项)
    - INSERT ... SELECT 会产生比 RBR 更多的行级锁
2. 基于行的复制(row-based replication, RBR)，
  - 优点：
    任何情况都可以被复制，这对复制来说是最安全可靠的
    多数情况下，从服务器上的表如果有主键的话，复制就会快了很多
    复制以下几种语句时的行锁更少：
    - INSERT ... SELECT
    - 包含 AUTO_INCREMENT 字段的 INSERT
    - 没有附带条件或者并没有修改很多记录的 UPDATE 或 DELETE 语句
    - 执行 INSERT，UPDATE，DELETE 语句时锁更少
    - 从服务器上采用多线程来执行复制成为可能。
  - 缺点：
    binlog 文件太大
    复杂的回滚时 binlog 中会包含大量的数据
    主服务器上执行 UPDATE 语句时，所有发生变化的记录都会写到 binlog 中，而 SBR 只会写一次，这会导致频繁发生 binlog 的并发写问题
    UDF 产生的大 BLOB 值会导致复制变慢
    无法从 binlog 中看到都复制了写什么语句，无法进行审计。

## Redis有哪些集群模式，各自的区别？

1. 主从复制，基于`SYNC`命令，如果主节点都挂了，服务也就挂了，不能水平扩容
2. 哨兵模式，基于状态监控，可选举出新的master节点，有一定弹性抗灾能力，不能水平扩容
3. Redis官方提供的Cluster集群模式(服务端)，基于数据分片
4. Jedis sharding集群(客户端sharding)，基于hash算法
5. 利用中间件代理，比如codis等，具体看使用的中间件

## 项目用到了多线程，如果线程数很多会怎么样？

> 这个问题属于一个比较开发性的问题，且能延伸出很多东西。
首先线程数过多，会导致CPU占用率居高不下，相应的散热需求就会上升，费电是一方面，如果占用率长时间居高不下很有可能导致服务器宕机。

> Q: 那么如何控制线程数？
> A: 线程池。消息队列。

> Q: 如何确定一个合理的线程数？
> A: 处理器敏感的程序，和CPU核心数持平；对于处理并发和网络通讯的程序，开几个到几十个，但不宜太多。

> Q: 如何处理大任务？
> A: 使用Fork/Join 框。

## 分布式了解哪些东西，消息队列了解么，用在什么场景？怎么保证Kafka数据不丢失，以及确保消息不会被重复消费？消息送达确认是怎么做的？

> 应用场景：削峰，限流和异步

> 消息队列 Kafka

关于kafka
使用同步模式的时候，有3种状态保证消息被安全生产，在配置为1（只保证写入leader成功）的话，如果刚好leader partition挂了，数据就会丢失。
还有一种情况可能会丢失消息，就是使用异步模式的时候，当缓冲区满了，如果配置为0（还没有收到确认的情况下，缓冲池一满，就清空缓冲池里的消息），
数据就会被立即丢弃掉。

在数据生产时避免数据丢失的方法：
只要能避免上述两种情况，那么就可以保证消息不会被丢失。
就是说在同步模式的时候，确认机制设置为-1，也就是让消息写入leader和所有的副本。
还有，在异步模式下，如果消息发出去了，但还没有收到确认的时候，缓冲池满了，在配置文件中设置成不限制阻塞超时的时间，也就说让生产端一直阻塞，这样也能保证数据不会丢失。

在数据消费时，避免数据丢失的方法：如果使用了storm，要开启storm的ackfail机制；如果没有使用storm，确认数据被完成处理之后，再更新offset值。低级API中需要手动控制offset值。

数据重复消费的情况，如果处理？
- 去重：将消息的唯一标识保存到外部介质中，每次消费处理时判断是否处理过；
- 不管：大数据场景中，报表系统或者日志信息丢失几条都无所谓，不会影响最终的统计分析结果。

Kafka本身支持At least once消息送达语义，因此实现消息发送的幂等关键是要实现Broker端消息的去重。为了实现消息发送的幂等性，Kafka引入了两个新的概念：

- **PID**:每个新的Producer在初始化的时候会被分配一个唯一的PID，这个PID对用户是不可见的。
- **Sequence Numbler**:对于每个PID，该Producer发送数据的每个`<Topic, Partition>`都对应一个从0开始单调递增的Sequence Number

> Broker端在缓存中保存了这Sequence Numbler，对于接收的每条消息，如果其序号比Broker缓存中序号大于1则接受它，否则将其丢弃。这样就可以实现了消息重复提交了。但是，只能保证单个Producer对于同一个<Topic, Partition>的Exactly Once语义。不能保证同一个Producer一个topic不同的partion幂等。

# 四面(HR面)

> 这里全靠自己发挥了

1. 工作中遇到的最大挑战是什么，你如何克服的？
2. 你最大的优点和最大的缺点，各自说一个？
> 缺点尽量说一些无关痛痒的，不要说自己不善于表达。
3. 未来的职业发展，短期和长期的规划是什么？
> 实在迷茫没想法，想一想自己是不是有心力去钻研技术，有就说技术晋升路线，没有就管理晋升路线，百度或Google上有很多模板可以借鉴。