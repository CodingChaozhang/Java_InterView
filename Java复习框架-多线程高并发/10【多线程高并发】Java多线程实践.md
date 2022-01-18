# 【多线程高并发】-Java多线程实践

## 一、Java两个线程轮流打印1-100两个数

```java
package com.lcz.thread;


// 多线程:两个线程轮流打印1-100
// 解题思路：synchronized以及wait和notify
//新建一个类，用于打印这个类
class Num{
	int i = 1;
	// 标识
	boolean flag = false; 
	public Num() {
		
	}
}
// 奇数线程
class PrintOdd implements Runnable{
	Num num;
	public PrintOdd(Num num) {
		this.num = num;
	}
	
	@Override
	public void run() {
		while(num.i<=100) {
			// 锁住该对象
			synchronized (num) {
				// 判断当前线程是否执行
				if(num.flag) {
					try {
						num.wait();
					}catch(InterruptedException e) {
						System.out.println(e.getMessage());
					}
				}else {
					System.out.println(Thread.currentThread().getName()+num.i);
					num.i++;
					num.flag = true;
					// 唤醒
					num.notify();
				}
				
			}
		}
		
	}
}
	
// 偶数线程
class PrintEven implements Runnable{
	Num num;
	public PrintEven(Num num) {
		this.num = num;
	}
	@Override
	public void run() {
		// TODO Auto-generated method stub
		while(num.i<=100) {
			// 锁住这个对象
			synchronized (num) {
				if(!num.flag) {
					try {
						num.wait();
					}catch (InterruptedException e) {
						// TODO: handle exception
						System.out.print(e.getMessage());
					}
				}else {
					System.out.println(Thread.currentThread().getName()+num.i);
					num.i++;
					num.flag = false;
					num.notify();
				}
			}
		}
	}
}
	
public class Print_Odd_Even {
	
	public static void main(String[] args) {
		Num num = new Num();
		
		PrintOdd printOdd = new PrintOdd(num);
		PrintEven printEven = new PrintEven(num);
		
		Thread thread1 = new Thread(printOdd);
		Thread thread2 = new Thread(printEven);
		
		thread1.setName("奇数线程:");
		thread2.setName("偶数线程:");
		thread1.start();
		thread2.start();
	}
	
}

```



> 更优雅的方法

```java
package com.lcz.thread;

import com.sun.org.apache.xalan.internal.xsltc.compiler.sym;

// 多线程:两个线程轮流打印1-100
// 解题思路：synchronized以及wait和notify
class MyThread{
	int num = 1;
	// 方法
	public void increase() {
		while(num<=100) {
			synchronized (this) {
				notify();
				System.out.println(Thread.currentThread().getName()+" "+(num++));
				try {
					wait();
				}catch (InterruptedException e) {
					System.out.print(e.getMessage());
				}
			}
		}
		
	}
}

public class Print_Odd_Even2 {
	
	public static void main(String[] args) {
		MyThread mythread = new MyThread();
		
		new Thread(()-> {
			mythread.increase();
		}).start();
		
		new Thread(()-> {
			mythread.increase();
		}).start();
	}
	
}

```



> 还可以用`ReentrantLock`和`Condition`来打印

```java
package com.lcz.thread;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
// 多线程:两个线程轮流打印1-100
// 解题思路：synchronized以及wait和notify
class MyThread{
	int num = 1;
	ReentrantLock lock = new ReentrantLock();
	Condition condition = lock.newCondition();
	
	public void increase() {
		while(num <= 100) {
			try {
				lock.lock();
				condition.signal();
				System.out.println(Thread.currentThread().getName() + " " + num++);
				try {
					condition.await();
				} catch (InterruptedException e) {
					e.printStackTrace();
				} 
			} finally {
				lock.unlock();
			}
		}
	}
}

public class Print_Odd_Even2 {
	
	public static void main(String[] args) {
		MyThread mythread = new MyThread();
		
		new Thread(()-> {
			mythread.increase();
		}).start();
		
		new Thread(()-> {
			mythread.increase();
		}).start();
	}
	
}

```



## 二、Java多线程轮流打印字符串

利用两个线程交替打印一个字符串“我爱北京天安门”

```java
package com.lcz.thread;

// 头条面试题之实现两个线程轮流打印字符串
class MyThreadV2{
	int i = 0;
	String str = "我爱北京天安门";
	
	public void print() {
		while(i<str.length()) {
			// 锁住
			synchronized (this) {
				// 唤醒
				notify();
				// 打印
				System.out.println(Thread.currentThread().getName()+" :"+ str.charAt(i++));
				// 等待
				try {
					wait();
				}catch(InterruptedException e){
					System.out.println(e.getMessage());
				}
			}
		}
	}
}

public class Print_Odd_Even3 {
	
	public static void main(String[] args) {
		MyThreadV2 myThread = new MyThreadV2();
		new Thread(()->{
			myThread.print();
		}).start();
		
		new Thread(()->{
			myThread.print();
		}).start();
	}
	
}

```

