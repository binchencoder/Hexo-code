---
title: synchronized和lock
date: 2019-09-05 21:01:08
tags:
    - 线程安全
    - synchronized
    - lock
categories:
    - 后端
---

# 开篇

Java 使用```synchronized```和```Lock```两种机制实现某种共享资源的同步

- **synchronized** 使用Object对象本身的notify、wait、notifyAll调度机制

- **Lock** 可以使用Condition(java.util.concurrent.locks.Condition)进行线程之间的调度，完成synchronized的所有功能

## 区别

### 用法不一样

在需要同步的对象中加入```synchronized```，synchronized既可以加在方法上，也可以加在特定的代码中，括号中表示需要锁的对象。而Lock需要显示的指定起始位置和终点位置。synchronized是托管给JVM执行的，而Lock的锁定是通过代码实现的，它有比synchronized 更精确的线程定义

### 性能不一样

在JDK1.5中增加的ReentrantLock, 它不仅拥有和synchronized相同的并发性和内存语义，还增加了锁投票，定时锁。等候和中断锁等。

他们的性能在不同的情况下会不同：在资源竞争不是很激烈的情况下，synchronized的性能要优于ReentantLock, 而在资源竞争很激烈的情况下， synchronized的性能会下降的比较快，而ReentantLock的性能基本保持不变

### 锁机制不一样

synchronized获得锁和释放锁的机制都在代码块中，当获得锁时，必须以相反的机制去释放，并且自动解锁，不会因为异常导致没有被释放而导致死锁。

Lock需要开发人员手动去释放，并且在finally代码块中，否则可能引起死锁问题。Lock提供了更强大的功能，可以通过tryLock的方式采用非阻塞的方式获取锁。