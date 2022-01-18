# 15【Java多体系】JUC之atomic框架之LongAdder.md

## 一、LongAdder简介

JDK1.8时，`java.util.concurrent.atomic`包中提供了一个新的原子类：`LongAdder`。
根据Oracle官方文档的介绍，LongAdder在高并发的场景下会比它的前辈————AtomicLong 具有更好的性能，代价是消耗更多的内存空间：

那么，问题来了：

> **为什么要引入`LongAdder`？ `AtomicLong`在高并发的场景下有什么问题吗？ 如果低并发环境下，`LongAdder`和`AtomicLong`性能差不多，那`LongAdder`是否就可以替代`AtomicLong`了？**

## 二、为什么要引入LongAdder？

我们知道，**AtomicLong**是利用了底层的CAS操作来提供并发性的，比如**addAndGet**方法：

![image-20210708160146073](E:\笔记\面试高频\imgs\403.png)

上述方法调用了**Unsafe**类的**getAndAddLong**方法，该方法内部是个**native**方法，它的逻辑是采用自旋的方式不断更新目标值，直到更新成功。

在并发量较低的环境下，线程冲突的概率比较小，自旋的次数不会很多。但是，高并发环境下，N个线程同时进行自旋操作，会出现大量失败并不断自旋的情况，此时**AtomicLong**的自旋会成为瓶颈。

这就是**LongAdder**引入的初衷——解决高并发环境下**AtomicLong**的自旋瓶颈问题。

## 三、LongAdder快在哪里？

既然说到**LongAdder**可以显著提升高并发环境下的性能，那么它是如何做到的？这里先简单的说下**LongAdder**的思路，第二部分会详述**LongAdder**的原理。

我们知道，**AtomicLong**中有个内部变量**value**保存着实际的long值，所有的操作都是针对该变量进行。也就是说，高并发环境下，value变量其实是一个热点，也就是N个线程竞争一个热点。

**LongAdder**的基本思路就是***分散热点\***，将value值分散到一个数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作，这样热点就被分散了，冲突的概率就小很多。如果要获取真正的long值，只要将各个槽中的变量值累加返回。

这种做法有没有似曾相识的感觉？没错，[ConcurrentHashMap](https://segmentfault.com/a/1190000015865714#)中的“分段锁”其实就是类似的思路。

总之，低并发、一般的业务场景下AtomicLong是足够了。如果并发量很多，存在大量写多读少的情况，那LongAdder可能更合适。适合的才是最好的，如果真出现了需要考虑到底用AtomicLong好还是LongAdder的业务场景，那么这样的讨论是没有意义的，因为这种情况下要么进行性能测试，以准确评估在当前业务场景下两者的性能，要么换个思路寻求其它解决方案。

## 四、LongAdder原理

之前说了，**AtomicLong**是多个线程针对单个热点值value进行原子操作。而**LongAdder**是每个线程拥有自己的槽，各个线程一般只对自己槽中的那个值进行CAS操作。

> 比如有三个ThreadA、ThreadB、ThreadC，每个线程对value增加10。

对于**AtomicLong**，最终结果的计算始终是下面这个形式：
![image-20210708160556740](E:\笔记\面试高频\imgs\404.png)

但是对于**LongAdder**来说，内部有一个`base`变量，一个`Cell[]`数组。
`base`变量：非竞态条件下，直接累加到该变量上
`Cell[]`数组：竞态条件下，累加个各个线程自己的槽`Cell[i]`中
最终结果的计算是下面这个形式：
![image-20210708160618929](E:\笔记\面试高频\imgs\405.png)









