# BTrace

BTrace是一个可以在不改代码、不重启应用的情况下，动态的查看程序运行细节的工具，堪称技术人员排查线上问题的神器（只要在JVM上运行的程序支持）。最近深入的研究了一下，分享出来给大家瞅瞅，欢迎大家拍砖。

BTrace介绍

先看官网是怎么介绍的：

BTrace is a safe, dynamic tracing tool for Java.

BTrace works by dynamically (bytecode) instrumenting classes of a running Java program.

BTrace inserts tracing actions into the classes of a running Java program and hotswaps the traced program classes.

 

如果大家觉得文档比较多，可以跳过文档部分，直接先看样例哦。



官方网站 http://kenai.com/projects/btrace

官方文档 http://kenai.com/projects/btrace/pages/UserGuide

中文翻译 http://macrochen.iteye.com/blog/838920 （伯岩）

相关介绍：

http://rdc.taobao.com/team/jm/archives/509  （毕玄）

http://www.slideshare.net/ykdsg/btrace-intro  （撒迦）



官方文档不是很多，讲解的也很清晰，更带有实例。



安装部署

1.     从官网下载最新版的btrace（http://kenai.com/projects/btrace/downloads/directory/releases ）

2.     解压到/home/admin/btrace-test（该路径可以自己决定）



BTrace命令

btrace [-I ] [-p ] [-cp ] []

·         -I 预编译路径，没有这个表明跳过预编译

·         include-path: 指定用来编译脚本的头文件路径(关于预编译可参考例子ThreadBean.java)

·         port : btrace agent端口, 默认是2020

·         classpath : 编译所需类路径, 一般是指btrace-client.jar等类所在路径（即/home/admin/btrace-test/btrace/build/）

·         pid : java进程id，可以通过固定命令获取：ps aux|grep "/home/admin"|grep java | awk '{print $2}'

·         btrace-script: btrace脚本, 如果是java文件, 则是未编译, class文件, 则是已编译过的，我们把所有的btrace类放到/home/admin/btrace-test/下统一管理

·         args: 传递给btrace脚本的参数, 在脚本中可以通过$(), $length()来获取这些参数(定义在BTraceUtils中)

还支持预编译（btracec命令），相见官方文档。



按照我们前面的设定，我们的btrace命令为：

cd /home/admin/btrace-test

./btrace/bin/btrace –cp ./btrace/build/ `ps aux|grep "/home/admin"|grep java | awk '{print $2}'`  ./XXX.java



完整的命令为：

/home/admin/btrace-test/btrace/bin/btrace –cp /home/admin/btrace-test/btrace/build/ `ps aux|grep "/home/admin"|grep java | awk '{print $2}'`  /home/admin/btrace-test/XXX.java







常用样例

      运行时代码（样例）：

public class BTraceCase {

   private static AtomicInteger counter = new AtomicInteger(0);

   private int total = 0;

   public int add(int count) throws Exception {

      if (count < 0) {

         throw new IllegalArgumentException("illegal argument count:" + count);

      }

      counter();

      return sum(count);

   }

   protected int sum(int count) {

      total += count;

      return total;

   }

   protected void counter() {

      counter.incrementAndGet();

   }

   public void execute(int count) {

      try {

         int reuslt = add(count);

         System.out.println(reuslt);

         Thread.sleep(2000);

      } catch (Exception e) {

         System.err.println(e);

      }

   }

   public static void main(String[] args) {

      Random random = new Random();

      BTraceCase obj = new BTraceCase();

      while (true) {

         obj.execute(random.nextInt(10) - 3);

      }

   }

}



Case1：方法进入时，@ProbeClassName取当前trace的类名，@ProbeMethodName取当前trace的方法名，count为参数

@BTrace

class BTraceTest {

   // 方法进入

   @OnMethod(clazz = "com.xxx.btrace.BTraceCase", method = "sum", location = @Location(Kind.ENTRY))

   void methodEntry(@ProbeClassName String pcn, int count, @ProbeMethodName String pmn) {

      print("entry ");

      print(pcn);

      print(".");

      print(pmn);

      print("(");

      print(count);

      println(")");

// eg:entry com.xxx.btrace.BTraceCase.sum(4)

   }

}



Case2：方法返回时，@Return取方法返回值

@BTrace

class BTraceTest {

   // 方法返回

   @OnMethod(clazz = "com.xxx.btrace.BTraceCase", method = "sum", location = @Location(Kind.RETURN))

   void methodReturn(@ProbeClassName String pcn, int count, @ProbeMethodName String pmn, @Return int sum) {

      print(pcn);

      print(".");

      print(pmn);

      print("(");

      print(count);

      print(") return ");

      println(str(sum));

   }

// eg:com.xxx.btrace.BTraceCase.sum(4) return 85

}



Case3： this对象，对象属性值获取，@Self取this对象，通过field和get组合可以获取this对象中的属性值

@BTrace

class BTraceTest {

   // this对象，对象属性值获取

   @OnMethod(clazz = "com.xxx.btrace.BTraceCase", method = "add", location = @Location(Kind.RETURN))

   void instanceAndProperty(@Self BTraceCase instance) {

      println(instance);

      println(strcat("total is:", str(get(field("com.xxx.btrace.BTraceCase", "total"), instance))));

      println(strcat("counter is:", str(get(field("com.xxx.btrace.BTraceCase", "counter"), instance))));

   }

// eg:com.xxx.btrace.BTraceCase@174d93a

// total is:77

// counter is:24

}



Case4：执行耗时，可通过@Duration获取方法执行的纳秒耗时

@BTrace

class BTraceTest {

   // 执行耗时

   @OnMethod(clazz = "com.xxx.btrace.BTraceCase", method = "counter", location = @Location(Kind.RETURN))

   void duration(@ProbeClassName String pcn, @ProbeMethodName String pmn, @Duration long duration) {

      print(pcn);

      print(".");

      print(pmn);

      print("() duration ");

      println(str(duration));

   }

// eg: com.xxx.btrace.BTraceCase.counter() duration 1642

}



Case5：执行耗时，可通过@Duration获取方法执行的纳秒耗时

@BTrace

class BTraceTest {

   // 方法被谁调用

   @OnMethod(clazz = "com.xxx.btrace.BTraceCase", method = "add", location = @Location(Kind.RETURN))

   void caller(@ProbeClassName String pcn, @ProbeMethodName String pmn) {

      print("who call ");

      print(pcn);

      print(".");

      print(pmn);

      println("():");

      jstack();

}

// eg: who call com.xxx.btrace.BTraceCase.add():

// com.xxx.btrace.BTraceCase.add(BTraceCase.java:25)

// com.xxx.btrace.BTraceCase.execute(BTraceCase.java:39)

// com.xxx.btrace.BTraceCase.main(BTraceCase.java:51)

}



Case6：方法抛出异常（Kind.THROW）或异常捕捉（Kind.CATCH）

@BTrace

class BTraceTest {

   // 方法抛出异常

   @OnMethod(clazz = "com.xxx.btrace.BTraceCase", method = "add", location = @Location(Kind.THROW))

   void throwException(@ProbeClassName String pcn, @ProbeMethodName String pmn, Throwable throwable) {

      print(pcn);

      print(".");

      print(pmn);

      print("(");

      print(") throw ");

      println(throwable);

   }

// eg: com.xxx.btrace.BTraceCase.add() throw java.lang.IllegalArgumentException: illegal argument count:-1



   // 异常捕捉

   @OnMethod(clazz = "com.xxx.btrace.BTraceCase", method = "execute", location = @Location(Kind.CATCH))

   void cacthException(@ProbeClassName String pcn, @ProbeMethodName String pmn) {

      print(pcn);

      print(".");

      print(pmn);

      print("(");

      println(") cacth Exception");

   }

// eg: com/xxx/btrace/BTraceCase.execute() cacth Exception

}



Case7：代码哪行执行

@BTrace

class BTraceTest {

   // 方法行执行

   @OnMethod(clazz = "com.xxx.btrace.BTraceCase", method = "execute", location = @Location(value = Kind.LINE, line = 40))

   void throwException(@ProbeClassName String pcn, @ProbeMethodName String pmn, int line) {

      print(pcn);

      print(".");

      print(pmn);

      print("() ");

      print(line);

      println(" line executed");

}

// eg: com.xxx.btrace.BTraceCase.execute() 40 line executed

}



 BTrace OnMethod方法仅仅是btrace定义的一种，它还支持OnTimer（定时执行trace方法），OnError（trace有异常抛出时执行），OnExit（trace调用内置exit方法时执行），OnEvent（捕获btraceclient触发的事件），OnLowMemory（内存低于给定阀值时执行）
