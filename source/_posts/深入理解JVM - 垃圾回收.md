---
title: 深入理解JVM - 垃圾回收
date: 2019-08-23 23:00:18
tags:
    - JVM
    - 垃圾回收
    - Java
categories:
    - 后端
---

# 前言

在 [JVM内存结构](https://binchencoder.github.io/2019/08/21/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3JVM%20-%20%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84/) 中我们详细讲解了JVM中的内存是如何分布和组成的。

我们已经知道JVM内存结构主要有三大块：```堆内存```、```方法区```和```栈内存```，而堆又是JVM中占用内存最大的一块，但是堆占用的空间也不是无限的(在JVM中会有参数来进行控制)，在程序运行的过程中，会不断的产生对象，导致堆的内存空间减少，这时我们会想到一定有一种机制来进行回收。接下来我们就一起来看看JVM是如何回收不再使用的对象空间的。


## 垃圾回收机制

#### Java垃圾回收主要关注堆

![image](https://upload-images.jianshu.io/upload_images/14923529-c0cbbccaa6858ca1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> Java内存运行时区域中的程序计数器、虚拟机栈、本地方法栈 都是线程私有的，因此它们的生命周期与线程的生命周期是一样的；栈中的栈帧随着方法的进入和退出有条无紊地执行着出栈和入栈操作。每一个栈帧中分配多少内存基本上是在类结构确定下来时就已知的（尽管在运行期会由JIT编译器进行一些优化）。因此这几个区域的内存分配和回收都具备确定性，不需要过多考虑回收的问题，因此方法结束或者线程结束时，内存自然就随着回收了。

而Java堆不一样，一个接口中的多个实现类需要的内存不一样，一个方法中的多个分支需要的内存也可能不一样，我们只有在程序处于运行期间时才能知道创建哪些对象，这部分内存的分配和回收都是动态的，垃圾收集器所关注的是这部分内存。

#### 判断需要被回收对象的方法

> **引用计数法**

该方法简单高效，缺点是无法解决对象之间相互循环引用的问题

> **可达性分析法**

![image](https://upload-images.jianshu.io/upload_images/2184951-06859aad2e07258d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决了```引用计数法``` 循环引用的问题

不可达的对象将暂时处于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历两次标记过程

## 垃圾收集算法

一共有四种：标记-清除算法、复制算法、标记-整理算法、分代收集算法

### 标记-清除算法(Mark-Sweep)

这是最基础的算法，分为“标记” 和 “清除” 两个阶段。首先**标记**出所有需要回收的对象，在标记完成后统一回收所有被标记的对象

**缺点：**
- 效率问题
- 空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多会导致以后需要分配较大的对象空间时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作

![image](https://upload-images.jianshu.io/upload_images/14923529-32d5034ee61de6cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 复制算法(Copying)

> **现在的商业虚拟机都采用这种算法来回收新生代**

复制算法是为了为了解决效率的问题，它将可用内存分为大小相等的两块。每次只使用其中的一块。当这一块的内存用完了，就将还活着的对象复制到另外一块上，然后把已使用的内存空间一次性清理掉。

这个算法的缺点是可用内存缩小为原来的一半

![image](https://upload-images.jianshu.io/upload_images/14923529-2f1313c6b35c1f5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 标记-整理算法(Mark-Compact)

复制算法在对象存活率较高的情况下要进行较多的复制操作，效率会降低。

复制算法并不适用于老年代，所以才有了```“标记-整理”``` 算法，标记的过程与```“标记-清除”``` 是一样的，第二步不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端便捷以外的内存

![image](https://upload-images.jianshu.io/upload_images/14923529-76fb92c61eff10b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 分代收集算法 (Generational Collection)

现代商业虚拟机的垃圾收集都采用这一算法，它是根据对象存活周期的不同将内存划分为几块并采用不同的垃圾收集算法。

一般将Java堆分为**新生代**和**老年代**，这样就可以根据各个年代的特点采用最适当的收集算法。
- 新生代：每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制法，只需要付出少量存活对象的复制成本就可以完成收集
- 老年代：因为对象存活率高，没有额外空间对它进行分配担保，那就必须使用**标记-清理** 或者 **标记-整理** 算法来进行回收

## 垃圾回收器分类

如果垃圾收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现

![image](https://upload-images.jianshu.io/upload_images/14923529-e4eb4c95e17d0a63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Serial收集器(串行收集器)

![image](https://upload-images.jianshu.io/upload_images/14923529-9bf73c0cdf9a7066.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这是一个单线程的收集器，它在进行垃圾收集时，必须暂停掉其他所有的工作线程，直到它收集结束（Stop The World）

### Serial Old收集器

![image](https://upload-images.jianshu.io/upload_images/14923529-7710a429f033fc31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是Serial收集器是老年代版本，也是一个单线程，使用 **标记-整理** 算法。

### ParNew收集器(串行收集器)

![image](https://upload-images.jianshu.io/upload_images/14923529-bdc22bf25c2c8ba7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> ParNew收集器是Serial收集器的多线程版本

除了使用多条线程进行垃圾收集之外，其余行为包括 Serial 收集器可用的所有控制参数（例如：-XX:SurvivorRatio、-XX:PretenureSizeThreshold、-XX:HandlePromotionFailure等）、收集算法、Stop The World、对象分配规则、回收策略等都与 Serial 收集器完全一样，在实现上，这两种收集器也共用了相当多的代码。

### Parallel Scavenge收集器(并行收集器)

这是一个新生代收集器，使用复制算法，又是并行的多线程收集器。Parallel Scavenge 收集器的目标是达到一个可控制的吞吐量(Througput)

> 吞吐量 = 运行用户代码时间 /（运行用户代码时间+垃圾收集时间），虚拟机总共运行了 100 分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%

### Parallel Old收集器(并行收集器)

![image](https://upload-images.jianshu.io/upload_images/14923529-627c4ce9f8eae23c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是Parallel Scavenge收集器的老年代版本，使用多线程和**标记-整理算法**

### CMS收集器(Concurrent Mark Sweep 并发标记扫描)

![image](https://upload-images.jianshu.io/upload_images/14923529-6401f1a80d3f1c81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是一种以获取最短回收停顿时间为目标的收集器

### G1收集器【重点】

![image](https://upload-images.jianshu.io/upload_images/14923529-8cdd3afd7d545435.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 垃圾回收器的选择


## References

- https://www.cnblogs.com/czwbig/p/11127159.html