# Java Performance

## Introduction

### 通过两方面的知识来提升JAVA平台的性能

 The goal is to give you an in-depth understanding of the performance aspects of the Java platform.

This knowledge falls into two broad categories:

* the way that the JVM is configured affects many aspects of a program’s performance

* understand how the features of the Java platform affect per‐
  formance

**Knowledge of the complete sphere is what will give your work the patina of art.**

### Software containers

* **Virtual matchine**: that virtual machine is indistinguishable from a regular machine with two cores and 16 GB of memory.

* **Docker container**: Java comes with a rich set of tools for diagnosing performance issues. These are often not available in a Docker container.

### Premeaturely Optimize

```java
log.log(Level.FINE, "I am here, and the value of X is " + calcX() + 
" and Y is " + calcY());
```

以上的代码没有达到应该打印的日志级别，calcX()和calcY()方法也会被执行。最好的方式使用下面这种写法：

```java
if (log.isLoggable(Level.FINE)) { 
    log.log(Level.FINE,"I am here, and the value of X is {} and Y is {}",
            new Object[]{calcX(), calcY()});
}
```

the message fomate isn't necessarily more ecfficient, but it is cleaner

### Look Elsewhere: The Database Is Always the Bottleneck

As a general rule, when load is increased into a system that is overburdened, performance of that system gets worse. If something is changed in the Java application that makes it more efficient—which only increases the load on an already overloaded database—overall performance may actually go down.

### Optimize for the Common Case

* 代码优化重点关注对代码的分析

* 使用奥卡姆剃刀原则诊断(diagnosing)性能问题，首先应该怀疑的是代码bug

* Write simple algorithms for the most common operations in an application

## An Approach to Performance Testing

### Microbenchmarks must use their results

使用递归方式的fib方法来测算执行时间

```java
public class SkipCalculation {

    public static void main(String[] args) {
        int l;
        long then = System.currentTimeMillis();
        for (int i = 0; i < 2; i++) {
             l = Sum.fib(20);
        }
        long now = System.currentTimeMillis();
        System.out.println("first execute time " + (now - then));

        long then1 = System.currentTimeMillis();
        for (int i = 0; i < 2; i++) {
            l = Sum.fib(20);
        }
        long now1 = System.currentTimeMillis();
        System.out.println("second execute time " + (now1 - then1));
    }
}
```

执行的结果：

> Task :SkipCalculation.main()
> first execute time 2
> second execute time 1

java code is interpreted the first few times it is executed, it gets faster the longer it is executed

这里的没有在任何的地方读取，会被编译器优化跳过计算，可以将l修改为成员变量，这样可以准确的测量该方法的性能。In practice, changing the definition of l from a local variable to an instance variable (declared with the volatile keyword) will allow the performance of the method to be measured.

### Microbenchmark code may behave differently in production

快和慢是相对的，有的代码实现快但是会造成大量的短暂存活的对象在新生代，并进入老年代，最后导致频繁的full GC。

### Mesobenchmarks

Mesobenchmarks are also good for autemated testing, particularly at the module level.

## A Java Performance ToolBox

### CPU Usage

* user time

* system time

CPU的利用率通常可以分为两种类型，user time和system time。user time CPU执行应用 代码，system time CPU执行系统内核代码。system time 关联应用，例如程序执行IO操作，将执行内核代码读取磁盘上的文件，或者将缓冲区的数据写到network。

CPU空闲的原因：

* 死锁

* 等待数据库的响应

* 应用无事可做

### Java Monitoring Tools

#### jcmd

> 打印class，thread，JVM的基本信息

常用命令：

*uptime*

```shell
jcmd process_id VM.updtime
```

The length of time the JVM has been up can be found via this command.

*System properties*

```shell
jcmd process_id VM.system.propertes
```

相当于System.getProperties()

*JVM command line* 

```shell
jcmd process_id VM.command_line
```

*JVM tuning flags*

```shell
jcmd process_id VM.flags
```

VM的启动参数

#### jconsole

> 图表展示JVM的活动状态，线程、class的使用，GC的活动状况

#### jmap

> 堆信息dumps，JVM内存使用

#### jinfo

> 展示JVM的系统设置

#### jstack

> dumps 栈信息

#### jstat

> 提供GC和类加载信息

#### jvisualvm

> JVM监控图形工具

### Working with tunning flags

`-XX:+PPrintFlagsFinal`

设置的VM参数可能会相互影响，开启该参数后可以打印出所有的设置的VM flags和对映的值。

## Working with the JIT Complier

just-in-time即时编译器是Java虚拟机的心脏，没有其他的东西对应用程序的性能影响超过了JIT Compiler。

compiled languages和interpreted的差异：

`compiled languages`:

* 不能跨平，一次编译永久执行, 二进制文件

* 执行快

`interpreted`:

* 解释器翻译为一个与平台无关的中间代码

* 运行时需要源代码

java尝试找到一个中间的解决方案。

### HostSpot Compilcation

JVM在执行代码的时候不会李哥变异代码，有以下两个原因：

* 代码可能只会被执行一次，那么编译它基本上就是一种浪费，直接解释java字节码的速度比编译后执行代码的速度快

* 被JVM多次执行的方法和循环，当代码被编译的时候，JVM会拥有足够的信息来优化。
  
  > 例如`equals()` 方法当解释其遇到b = obj1.equals(obje2) 语句的时候，先要查找obj1的类型，才能知道哪一个equals()方法被执行。这种动态查找是很耗时的。通常JVM注意到每次改语句执行的时候obj1是String类型，JVM可以在编译的时候直接调用String的equals()方法。

-XX:-TieredCompilation使用分层编译参数。

### Common Compiler Flags

调校Code Cache参数

> JVM编译代码后，会将会变代码存入code cache，一旦code cache装满，JVM就不能再编译任何的代码。JVM会先预留code cache设置的内存空间不分配，但是会被预留。extra memory reservation will generally be accepted by the operating system.

### 编译代码的几种状态

`%`: OSR: on-stack replacement。当JVM确定了需要编译的方法，这个方法在队列中被替换。

`s`: 同步方法

`!`: The method has an exception handler.

`b`: 编译在阻塞模式下进行

`n`: 编译包装一个native 方法

### 分成编译的层级

`0`: Interpreted code

`1`: Simple C1 compiled code

`2`: Limited C1 compiled code

`3`: Full C1 compile code

`4`: C2 compile code

通常编译日志显示大多数方法编译的等级在level3。所有的方法开始都在level0,但是不会显示日志。如果一个方法京城的被执行，将会提升编译等级到level4，最常见的是，C1 compiler等待需要编译的代码直到有了如何使用这些代码的信息，利用这些信息来执行优化。

如果C2的队列满了，需要编译的方法将从C2的队列中拉去出来使用level2等级编译。

### 高级的编译器参数

**编译阀值**

> 主要因素是代码执行的频率，一旦达到了一定的执行次数就达到了编译阀值，编译器就认为有足够的信息编译代码

在当前的JVM中修改编译的阀值没有任何的作用。

在JVM中编译器基于两个计数器

* 方法被调用的次数

* 方法中循环向后分支的次数the number of times any loops in the method have branched back（branceing counter分支计数？）

当JVM执行代码的时候会检查这两个计数器的和来决定是否编译

修改编译阀值的参数在分层编译器中是失效的，在standard compilation可以通过`-XX:CompileThreshold=N`来修改。

**编译器线程数**

当一个方法达到了编译的要求，改方法会加入一个队列，这个队列被一个或多个后台的线程处理。这个队列不是严格的先进先出的队列，被执行次数高的方法具有较高的优先级。C1和C2编译器用不同的队列，也有不同的线程对其进行处理。

`-XX:CICompilerCount=N`调整编译线程的数量，设置编译线程数量的三分之一会被JVM用于处理C1编译的队列，剩余的线程会用于处理C2编译器的队列。CICompilerCount的默认值是基于CPU的数量，低版本的JDK在容器中使用需要注意。
