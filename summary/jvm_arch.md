# JVM架构
JVM的整体架构图如下：

![JVM整体架构](http://s7.sinaimg.cn/orignal/721770d9gc8718e2f4066&690)

JVM架构包括两个子系统和两个组件，ClassLoader （类装载）子系统和Execution Engine（执行引擎）子系统，Native Interface （本地接口）组件和Runtime Data Area （运行时数据区域）组件。

## 两个子系统
ClassLoader （类装载）子系统根据全限定类名装载class文件的内容Method Area(方法区域)，且可继承ClassLoader类实现自定义Class loader。

Execution Engine（执行引擎）子系统主要的作用就是执行classes中的指令（机器码），它是任何JDK的核心，每一个JVM线程都有一个执行引擎实例。

 

## 两个组件
Native Interface （本地接口）组件用于与Native Libraries交互，是与其他编程语言交互的接口。

Runtime Data Area （运行时数据区域）组件包括Heap （堆）、Java Stack （栈）、 Method Area （方法区）、Native Method Stack （本地方法栈）以及Programe Counter （程序计数器）。
