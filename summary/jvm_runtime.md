# JVM运行时数据区域
整体架构图如下：

![JVM运行时数据区域](http://s2.sinaimg.cn/large/721770d9gc871c1fc9531&690)


Runtime Data Area （运行时数据区域）组件包括Heap （堆）、Java Stack （栈）、 Method Area （方法区）、Native Method Stack （本地方法栈）以及Programe Counter （程序计数器）。

## Heap（堆）
Heap （堆）用来存放运行时所有类实例或数组。

每个JVM实例只有一个Heap。

多线程访问对象（堆数据）同步问题。

每个Java程序占一个JVM实例。

每个Java程序有独立Heap，相互不影响。

异常溢出： OutOfMemoryError: Java Heap Space。

## Method Area（方法区）
 Method Area （方法区）存储被装载的class信息（过程详见类装载），包括一下信息：

1）static静态变量和静态方法

2）Method Code（方法代码）

3）Runtime Constant Pool（常量池）存常量

Method Area的线程共享和Heap一致

异常溢出： OutOfMemoryError: PermGen full。


## Java Stack（栈）
 Java Stack （栈）是线程独享的。

栈以帧为单位保存线程运行状态。

当线程调用方法，当前状态以帧单位压栈。

当方法调用返回，帧出栈，恢复调用前状态

存储局部变量、操作数、动态链接、方法出口。

帧中分配的局部变量内存在编译期完全确定。

栈大小有限制时，可能异常溢出：StackOverFlowError。

JVM参数：-Xss=512（可默认）。

## Native Method Stack（本地方法栈）
 Native Method Stack （本地方法栈）也是线程独享的。

本地方法不受虚拟机限制，它就像操作系统系统调用一样，可以通过本地方法接口，访问虚拟机运行数数据区，包括操作PC寄存器、分配系统内存等。

本地方法具有和JVM相同的能力和权限。

异常溢出：Native Heap OutOfMemoryError。


## Program Counter（PC寄存器）
 Program Counter （PC寄存器）指向的是当前线程执行的字节码行号。

它的内容总是指向下一条将被执行的指令的地址：可以是本地指针，也可以是方法的偏移量。

PC寄存器也是线程独享的，在线程启动时创建。

PC寄存器完全由JVM调度，程序不可调度。

PC寄存器是运行时数据区域唯一一个没有OutOfMemory异常溢出可能的区域。
