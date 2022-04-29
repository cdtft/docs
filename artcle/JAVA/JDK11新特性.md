# JDK11新特性

## 性能提升

### 动态文件常量

> Java class-file format is extended to support a new constant-pool form named *CONSTANT_Dynamic*.
> 
> Loading the new constant-pool will delegate creation to a bootstrap method, just as linking an *invokedynamic* call site delegates linkage to a bootstrap method.

`constant_pool` cp_info数据结构:

```c
struct cp_info {
    u1 tag;
    u1* info
}
```

* tag 表示不同的常量类型

* info 储存常量的值或引用

有了invokedynamic以后，由于invokedynamic bootstrap的静态参数列表是一系列常量，也就是好多个常量。在常量池中存储复杂数据的情况就变得比较普遍，**所以就需要一个更丰富、更高级的常量类型，这样可以解决很多开发invokedynamic协议的麻烦，从而还能改善程序性能和简化编译器逻辑**

### Improved Aarch64 Intrinsics

### A No-Op Garbage Collector

No-Op全称no operation，负责分配内存但是不做垃圾回收，Epsilon垃圾回收器。不适用于典型的java应用，使用场景如下：

- Performance testing
- Memory pressure testing
- VM interface testing and
- Extremely short-lived jobs（短暂的任务）

In order to enable it, use the  ***-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC*** flag.
