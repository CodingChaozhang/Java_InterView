# 【Java锁体系】-阻塞队列的实现

## 一、阻塞队列介绍

在新增的Concurrent包中，BlockingQueue很好地解决了多线程中，如何高效安全传输数据的问题。通过这些高效并且线程安全的队列类，为我们快速搭建高质量的多线程程序带来极大的便利。

**阻塞队列与普通队列的区别**在于，当队列是空的时，从队列中获取元素的操作会被阻塞，或者当队列是满时，往队列里添加元素的操作会被阻塞。试图从空的阻塞队列中获取元素的线程将会被阻塞，直到其它的线程往空的队列里面插入新的元素。同样，试图往已满的阻塞队列中添加新元素的线程同样也会被阻塞，直到其它的线程使队列重新变得空闲起来，如从队列中移除一个或者多个元素，或者完全清空队列。

## 二、Java中阻塞队列的实现

| 队列                  | 有界性             | 锁   | 数据结构   |
| --------------------- | ------------------ | ---- | ---------- |
| ArrayBlockingQueue    | bounded(有界)      | 加锁 | arrayList  |
| LinkedBlockingQueue   | optionally-bounded | 加锁 | linkedList |
| PriorityBlockingQueue | unbounded          | 加锁 | heap       |
| DelayQueue            | unbounded          | 加锁 | heap       |
| SynchronousQueue      | bounded            | 加锁 | 无         |
| LinkedTransferQueue   | unbounded          | 加锁 | heap       |
| LinkedBlockingDeque   | unbounded          | 无锁 | heap       |

- ArrayBlockingQueue：是一个用数组实现的有界阻塞队列，此队列按照先进先出（FIFO）的原则对元素进行排序。支持公平锁和非公平锁。【注：每一个线程在获取锁的时候可能都会排队等待，如果在等待时间上，先获取锁的线程的请求一定先被满足，那么这个锁就是公平的。反之，这个锁就是不公平的。公平的获取锁，也就是当前等待时间最长的线程先获取锁】。参考：ArrayBlockingQueue源码

- LinkedBlockingQueue：一个由链表结构组成的有界队列，此队列的长度为Integer.MAX_VALUE。此队列按照先进先出的顺序进行排序。参考：LinkedBlockingQueue源码
- PriorityBlockingQueue： 一个支持线程优先级排序的无界队列，默认自然序进行排序，也可以自定义实现compareTo()方法来指定元素排序规则，不能保证同优先级元素的顺序。参考：PriorityBlockingQueue源码
  

## 三、自己实现一个阻塞队列

在自己实现之前先搞清楚阻塞队列的几个特点：

- 基本队列特性：先进先出；
- 写入队列空间不可用时会阻塞；
- 获取队列数据时当队列为空时将阻塞；

实现队列的方式多种，总的来说就是数组和链表；不同的特性主要表现为数组的链表的区别。

### 3.1 队列所需的变量以及函数

```java
// 队列数量
	private int count = 0;
	// 最终的数据存储
	private Object[] items;
	
	// 队列满时的阻塞锁
	private Object full = new Object();
	// 队列空时的阻塞锁
	private Object empty = new Object();
	
	//写入数据时的下标
	private int putIndex;
	// 获取数据时的下标
	private int getIndex;
	
	// 初始化
	public ArrayQueue(int size) {
		items = new Object[size];
	}
```

### 3.2 写入队列

写入队列比较简单，只需要依次把数据存放到这个数组中即可；

但需要有两个注意的点：

- 队列满的时候，写入的线程需要被阻塞；
- 写入多队列的数量大于队列大小时需要从第一个下标开始写；

```java
/**
	 * put方法 从队列尾写入数据
	 */
	public void put(T t) {
		// 数据满的情况下
		synchronized (full) {
			while(count==items.length) {
				try {
					// 满了的情况下等待
					full.wait();
				}catch (InterruptedException e) {
					// TODO: handle exception
					break;
				}
			}
		}
		// 数据空的情况下
		synchronized (empty) {
			// 写入
			items[putIndex++] = t;
			count++;
			
			// 查看是否到尾巴了
			if(putIndex==items.length) {
				// 超过数组长度后需要从头开始
				putIndex++;
			}
			
			// 有数据了唤醒
			empty.notify();
		}	
	}
```

所以这里声明了两个对象用于队列满、空情况下的互相通知作用。

在写入数据成功后需要使用 `empty.notify()`，这样的目的是当获取队列为空时，一旦写入数据成功就可以把消费队列的线程唤醒。

### 3.3 消费队列

上文也提到了：当队列为空时，获取队列的线程需要被阻塞，直到队列中有数据时才被唤醒。

```java
// 从队列头获取数据
	public T get() {
		// 如果是空的情况
		synchronized (empty) {
			while(count==0) {
				try {
					empty.wait();
				}catch (InterruptedException e) {
					// TODO: handle exception
					return null;
				}
			}
		}
		
		// 如果是满的情况下
		synchronized (full) {
			Object result = items[getIndex];
			items[getIndex] = null;
			getIndex--;
			count--;
			
			if(getIndex==items.length) {
				getIndex=0;
			}
			// 唤醒线程
			full.notify();
			return (T)result;
		}
	}
```

![img](https://i.loli.net/2019/04/29/5cc654e141818.jpg)



总的来说就是：

- 写入队列满时会阻塞直到获取线程消费了队列数据后唤醒**写入线程**。
- 消费队列空时会阻塞直到写入线程写入了队列数据后唤醒**消费线程**。

### 3.4 测试

```java
package com.lcz.tencent.thread;

import org.junit.Test;

import com.sun.istack.internal.logging.Logger;


public class TestArrayQueue {
	// 测试单线程的写入和消费
		@Test
		public void put1() {
			final ArrayQueue<String> queue = new ArrayQueue<String>(3);
			Logger logger = Logger.getLogger(ArrayQueue.class);
			//开启一个线程来获取队列中的值
			new Thread(()->{
				try {
					logger.info(Thread.currentThread().getName() + "_" + queue.get());
				}catch (Exception e) {
					// TODO: handle exception
					e.printStackTrace();
				}
			}) .start();
			
			// 往队列里面放值
			queue.put("123");
			queue.put("1234");
			queue.put("12345");
		}

}

```

```java
@Test
		public void put2() throws InterruptedException {
			final ArrayQueue<String> queue = new ArrayQueue<String>(300);
			Logger logger = Logger.getLogger(ArrayQueue.class);
			// 开辟线程
			Thread t1 = new Thread(()->{
				for(int i=0;i<100;i++) {
					queue.put(i+"");
				}
			});
			Thread t2 = new Thread(()->{
				for(int i=0;i<100;i++) {
					queue.put(i+"");
				}
			});
			Thread t3 = new Thread(()->{
				for(int i=0;i<100;i++) {
					queue.put(i+"");
				}
			});
			
			// 取出来一个
			Thread t4 = new Thread(()->{
				System.out.println("==="+queue.get());
			});
			
			t1.start();
			t2.start();
			t3.start();
			t4.start();
			
			t1.join();
			t2.join();
			t3.join();
			
			System.out.println(queue.size());
		}
```

## 四、实际案例

背景任务：有一个定时任务会按照一定的间隔时间从数据库中读取一批数据，需要对这些数据做校验的同时调用一个远程接口。

简单的做法就是由这个定时任务的线程去完成读取数据、消息校验、调用接口等整个全流程；但这样会有一个问题：

假设调用外部接口出现了异常、网络不稳导致耗时增加就会造成整个任务的效率降低，因为他都是串行会互相影响。

![img](https://i.loli.net/2019/05/08/5cd1bedc34bcb.jpg)

其实就是一个典型的生产者消费者模型：

- 生产线程从数据库中读取消息丢到队列里。
- 消费线程从队列里获取数据做业务逻辑。

这样两个线程就可以通过这个队列来进行解耦，互相不影响，同时这个队列也能起到缓冲的作用。









