---
title: Kafka是如何实现几十万的高并发写入
date: 2019-08-28 10:23:58
tags:
	- MQ
	- Java
	- Kafka
categories:
	- 后端
---

# 开篇

在[初识kafka](https://binchencoder.github.io/2019/08/27/%E5%88%9D%E8%AF%86kafka/) 一文中讲了使用`MQ(消息队列)`来设计系统带来的好处：`业务解耦`、`流量削峰`、`灵活扩展`

当下流行的MQ有很多，因为我们公司在技术选型上选择了使用Kafka，所以我就整理了一篇关于Kafka的入门知识。通过[技术选型](https://binchencoder.github.io/2019/08/27/%E5%88%9D%E8%AF%86kafka/#%E6%8A%80%E6%9C%AF%E9%80%89%E5%9E%8B) 我们对业界主流的MQ进行了对比，Kakfa最大的优点就是**吞吐量高** 。

> Kafka是高吞吐低延迟的高并发、高性能的消息中间件，在大数据领域有极为广泛的运用。配置良好的Kafka集群甚至可以做到每秒几十万、上百万的超高并发写入。

那么Kafka是如何做到这么高的吞吐量和性能的呢？在入门之后我们就来深入的扒一下Kafka的架构设计原理，掌握这些原理在互联网面试中会占据优势

# 持久化

Kafka对消息的存储和缓存依赖于文件系统，每次接收数据都会往磁盘上写，人们对于**“磁盘速度慢”**的普遍印象，使得人们对于持久化的架构能够提供强有力的性能产生怀疑。

事实上，磁盘的速度比人们预期的要慢的多，也快得多，这取决于人们使用磁盘的方式。而且设计合理的磁盘结构通常可以和网络一样快。

![顺序磁盘访问在某些情况下比随机内存访问还要快](./kafka是如何实现几十万的高并发写入/持久化.jpg)

> 通过上图对比，我们可以看出实际上顺序磁盘访问在某些情况下比随机内存访问还要快，其实Kafka就是利用这一优势来实现高性能写磁盘

> **See** [http://kafka.apachecn.org/documentation.html#persistence](http://kafka.apachecn.org/documentation.html#persistence)

## 页缓存技术 + 磁盘顺序写

Kafka 为了保证磁盘写入性能，首先Kafka是基于操作系统的页缓存来实现文件写入的。

操作系统本身有一层缓存，叫做`page cache`，是在内存里的缓存，我们也可以称之为`os cache`，意思就是操作系统自己管理的缓存。

你在写磁盘文件的时候，可以直接写入`os cache` 中，也就是仅仅写入内存中，接下来由操作系统自己决定什么时候把`os cache` 里的数据真的刷入到磁盘中。

![kafka写磁盘](./kafka是如何实现几十万的高并发写入/kafka写磁盘.png)

> 通过上图这种方式可以将磁盘文件的写性能提升很多，其实这种方式相当于写内存，不是在写磁盘

### **顺序写磁盘**

另外还有非常关键的一点，Kafka在写数据的时候是以磁盘顺序写的方式来落盘的，也就是说，仅仅将数据追加到文件的末尾(append)，而不是在文件的随机位置来修改数据。

对于普通的机械硬盘如果你要是随机写的话，确实性能极低，这里涉及到磁盘寻址的问题。但是如果只是追加文件末尾按照顺序的方式来写数据的话，那么这种磁盘顺序写的性能基本上可以跟写内存的性能本身是差不多的。

> **来总结一下：** Kafka就是基于`页缓存技术 + 磁盘顺序写` 技术实现了写入数据的超高性能。
>
> 所以要保证每秒写入几万甚至几十万条数据的核心点，就是尽最大可能提升每条数据写入的性能，这样就可以在单位时间内写入更多的数据量，提升吞吐量。

## 零拷贝技术(zero-copy)

说完了写入这块，再来谈谈消费这块。

大家应该都知道，从Kafka里我们经常要消费数据，那么消费的时候实际上就是要从kafka的磁盘文件里读取某条数据然后发送给下游的消费者，如下图所示：

![kafka读磁盘](./kafka是如何实现几十万的高并发写入/kafka读磁盘.png)

如果Kafka以上面这种方式从磁盘中读取数据发送给下游的消费者，大概过程是：

1. 先看看要读的数据在不在`os cache`中，如果不在的话就从磁盘文件里读取数据后放入os cache
2. 接着从操作系统的`os cache` 里拷贝数据到应用程序进程的缓存里，再从应用程序进程的缓存里拷贝数据到操作系统层面的Socket缓存里，最后从Soket缓存里提取数据后发送到网卡，最后发送出去给下游消费者

整个过程如下图：

![kafka消费过程](./kafka是如何实现几十万的高并发写入/kafka消费过程.png)

> **从上图可以看出，这整个过程有两次没必要的拷贝**
>
> 一次是从操作系统的cache里拷贝到应用进程的缓存里，接着又从应用程序缓存里拷贝回操作系统的Socket缓存里。
>
> 而且为了进行这两次拷贝，中间还发生了好几次上下文切换，一会儿是应用程序在执行，一会儿上下文切换到操作系统来执行。
>
> 所以这种方式来读取数据是比较消耗性能的。

Kafka 为了解决这个问题，在读数据的时候是引入零拷贝技术。

也就是说，直接让操作系统的cache中的数据发送到网卡后传出给下游的消费者，中间跳过了两次拷贝数据的步骤，Socket缓存中仅仅会拷贝一个描述符过去，不会拷贝数据到Socket缓存。

![kafka零拷贝](./kafka是如何实现几十万的高并发写入/kafka零拷贝.png)

> **体会一下这个精妙的过程吧：**
>
> 通过零拷贝技术，就不需要把os cache里的数据拷贝到应用缓存，再从应用缓存拷贝到Socket缓存了，两次拷贝都省略了，所以叫做零拷贝。
>
> 对Socket缓存仅仅就是拷贝数据的描述符过去，然后数据就直接从os cache中发送到网卡上去了，这个过程大大的提升了数据消费时读取文件数据的性能。
>
> 而且大家会注意到，在从磁盘读数据的时候，会先看看os cache内存中是否有，如果有的话，其实读数据都是直接读内存的。
>
> 如果kafka集群经过良好的调优，大家会发现大量的数据都是直接写入os cache中，然后读数据的时候也是从os cache中读。
>
> 相当于是Kafka完全基于内存提供数据的写和读了，所以这个整体性能会极其的高。

# 总结

通过学习Kafka的优秀设计，我们了解了Kafka底层的页缓存技术的使用，磁盘顺序写的思路，以及零拷贝技术的运用，才能使得Kafka有那么高的性能，做到每秒几十万的吞吐量。

# 名词解释

- [吞吐量(TPS)](https://baike.baidu.com/item/%E5%90%9E%E5%90%90%E9%87%8F)：吞吐量是指对网络、设备、端口、虚电路或其他设施，单位时间内成功地传送[数据](https://baike.baidu.com/item/数据/5947370)的数量（以[比特](https://baike.baidu.com/item/比特/3431582)、[字节](https://baike.baidu.com/item/字节/1096318)、分组等测量）

# References

- http://kafka.apachecn.org/documentation.html#maximizingefficiency
- http://kafka.apachecn.org/documentation.html#persistence