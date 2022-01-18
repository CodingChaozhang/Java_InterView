# 伪共享

## 1.CPU架构

下图是计算的基本结构。L1、L2、L3分别表示一级缓存、二级缓存、三级缓存，越靠近CPU的缓存，速度越快，容量也越小。所以L1缓存很小但很快，并且紧靠着在使用它的CPU内核；L2大一些，也慢一些，并且仍然只能被一个单独的CPU核使用；L3更大、更慢，并且被单个插槽上的所有CPU核共享；最后是主存，由全部插槽上的所有CPU核共享。

![计算机CPU与缓存示意图](https://img-blog.csdnimg.cn/img_convert/88925a861ca8ec89508d96e31105d0f0.png)

级别越小的缓存，越接近CPU， 意味着速度越快且容量越少。

L1是最接近CPU的，它容量最小，速度最快，每个核上都有一个L1 Cache(准确地说每个核上有两个L1 Cache， 一个存数据 L1d Cache， 一个存指令 L1i Cache)；

L2 Cache 更大一些，例如256K，速度要慢一些，一般情况下每个核上都有一个独立的L2 Cache；二级缓存就是一级缓存的缓冲器：一级缓存制造成本很高因此它的容量有限，二级缓存的作用就是存储那些CPU处理时需要用到、一级缓存又无法存储的数据。

L3 Cache是三级缓存中最大的一级，例如12MB，同时也是最慢的一级，在同一个CPU插槽之间的核共享一个L3 Cache。三级缓存和内存可以看作是二级缓存的缓冲器，它们的容量递增，但单位制造成本却递减。

L3 Cache和L1，L2 Cache有着本质的区别。，L1和L2 Cache都是每个CPU core独立拥有一个，而L3 Cache是几个Cores共享的，可以认为是一个更小但是更快的内存

## 2.缓存行 cache line

缓存，是由缓存行Cache Line组成的。一般一行缓存行有64字节。所以使用缓存时，并不是一个一个字节使用，而是一行缓存行、一行缓存行这样使用；换句话说，CPU存取缓存都是按照一行，为最小单位操作的。

Cache Line可以简单的理解为CPU Cache中的最小缓存单位。目前主流的CPU Cache的Cache Line大小都是64Bytes。假设我们有一个512字节的一级缓存，那么按照64B的缓存单位大小来算，这个一级缓存所能存放的缓存个数就是512/64 = 8个。

## 3.伪共享问题

CPU的缓存系统是以缓存行(cache line)为单位存储的，一般的大小为64bytes。在多线程程序的执行过程中，存在着一种情况，多个需要频繁修改的变量存在同一个缓存行当中。

假设：有两个线程分别访问并修改X和Y这两个变量，X和Y恰好在同一个缓存行上，这两个线程分别在不同的CPU上执行。那么每个CPU分别更新好X和Y时将缓存行刷入内存时，发现有别的修改了各自缓存行内的数据，这时缓存行会失效，从L3中重新获取。这样的话，程序执行效率明显下降。为了减少这种情况的发生，其实就是避免X和Y在同一个缓存行中，可以主动添加一些无关变量将缓存行填充满，比如在X对象中添加一些变量，让它有64 Byte那么大，正好占满一个缓存行。

![img](https://img-blog.csdn.net/20160602150616280)

两个线程（Thread1 和 Thread2）同时修改一个同一个缓存行上的数据 X Y:

如果线程1打算更改a的值，而线程2准备更改b的值：

```java
Thread1：x=3;

Thread2：y=2;
```

由x值被更新了，所以x值需要在线程1和线程2之间传递（从线程1到线程2），x的变更会引起整块64bytes被交换，因为cpu核之间以cache lines的形式交换数据(cache lines的大小一般为64bytes)。

每个线程在不同的核中被处理。x,y是两个频繁修改的变量，x,y，还位于同一个缓存行.

如果，CPU1修改了变量x时，L3中的缓存行数据就失效了，也就是CPU2中的缓存行数据也失效了，CPU2需要的y需要重新从内存加载。

如果，CPU2修改了变量y时，L3中的缓存行数据就失效了，也就是CPU1中的缓存行数据也失效了，CPU1需要的x需要重新从内存加载。

x,y在两个cpu上进行修改，本来应该是互不影响的，但是由于缓存行在一起，导致了相互受到了影响。

伪共享问题（False Sharing） **全网最清晰的解答**：

**对缓存行中的单个变量进行修改了，导致整个缓存行数据也就失效了，并且，会导致其他CPU的含有被改动的共享变量的缓存行也失效了，就是所谓的伪共享问题。**

而缓存行是为了解决缓存一致性的问题，所引入的。所以，解决缓存一致性的问题，又导致了伪共享问题的出现。

## 4.伪共享问题的解决方案

简单的说，就是 以**空间换时间**： 使用占位字节，将变量的所在的 缓冲行 塞满。

disruptor 无锁框架就是这么干的。

## 5.缓冲和塞满的例子

下面是一个填充了的缓存行的，尝试 p1, p2, p3, p4, p5, p6为AtomicLong的value的缓存行占位，将AtomicLong的value变量的所在的 缓冲行 塞满，

**代码如下:**

~~~java
package com.baidu.fsg.uid.utils;

//...省略import
public class PaddedAtomicLong extends AtomicLong {
    private static final long serialVersionUID = -3415778863941386253L;

```
/** Padded 6 long (48 bytes) */
public volatile long p1, p2, p3, p4, p5, p6 = 7L;

/**
 * Constructors from {@link AtomicLong}
 */
public PaddedAtomicLong() {
    super();
}

public PaddedAtomicLong(long initialValue) {
    super(initialValue);
}

/**
 * To prevent GC optimizations for cleaning unused padded references
 */
public long sumPaddingToPreventOptimjavaization() {
    return p1 + p2 + p3 + p4 + p5 + p6;
}
```

}
~~~

**例子的部分结果如下：**

```java
printable = com.crazymakercircle.basic.demo.cas.busi.PaddedAtomicLong object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           50 08 01 f8 (01010000 00001000 00000001 11111000) (-134150064)
     12     4        (alignment/padding gap)                  
     16     8   long AtomicLong.value                          0
     24     8   long PaddedAtomicLong.p1                       0
     32     8   long PaddedAtomicLong.p2                       0
     40     8   long PaddedAtomicLong.p3                       0
     48     8   long PaddedAtomicLong.p4                       0
     56     8   long PaddedAtomicLong.p5                       0
     64     8   long PaddedAtomicLong.p6                       7
Instance size: 72 bytes
Space losses: 4 bytes internal + 0 bytes external = 4 bytes total
```

