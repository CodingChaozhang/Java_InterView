# 【大话Java面试】-如何通俗易懂的理解单例模式？

**单例模式定义：一个类有且仅有一个实例，并且自行实例化向整个系统提供。**

其实现方式主要是通过饿汉式（线程安全的）和懒汉式（线程不安全的）实现方式。

## 1.饿汉式的实现方式

```java
package com.lcz.syn;

// 单例模式实现方式-饿汉式
class SingleOne{
   // 创建类中私有构造
    private SingleOne(){

    }
    // 创建私有对象
    private static SingleOne instance = new SingleOne();

    // 创建公有静态返回
    public static SingleOne getInstance(){
        return instance;
    }
}

public class Test1 {
    // 主函数
    public static void main(String[] args){
        SingleOne singleOne1 = SingleOne.getInstance();
        SingleOne singleOne2 = SingleOne.getInstance();
        System.out.println(singleOne1 == singleOne2);
    }
}

```

## 2.懒汉式的实现方式



```java
// 单例模式实现方式-懒汉式
class SingeleTwo{
    // 创建类中私有构造
    private SingeleTwo(){

    }
    // 创建静态私有对象
    private static SingeleTwo instance;
    // 创建返回
    public static SingeleTwo getInstance(){
        if (instance==null){
            instance = new SingeleTwo();
        }
        return instance;
    }
}

public class Test1 {
    // 主函数
    public static void main(String[] args){
        SingeleTwo singleOne1 = SingeleTwo.getInstance();
        SingeleTwo singleOne2 = SingeleTwo.getInstance();
        System.out.println(singleOne1 == singleOne2);
    }
}
```

但是懒汉式存在的问题是：在多线程的环境下是不安全的，可能创建等多个实例对象。

通过是用双重校验锁的方式使其安全。

## 3.双重校验锁实现懒汉式

```java
// 双重校验锁实现
class Singleton{
    //私有化
    private Singleton(){

    }
    // 私有化对象
    private volatile static Singleton instance;

    // 返回方法
    public static Singleton getInstance(){
        if (instance==null){
            // 加锁
            synchronized (Singleton.class){
                if (instance==null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

}
```

**1.为什么使用volatile修饰了singleton引用还用synchronized？**

volatile 只保证了共享变量 singleton 的可见性，但是 `singleton = new Singleton();` 这个操作不是原子的，可以分为三步：

步骤1：在堆内存申请一块内存空间；

步骤2：初始化申请好的内存空间；

步骤3：将内存空间的地址赋值给 singleton；

所以`singleton = new Singleton();` 是一个由三步操作组成的复合操作，多线程环境下A 线程执行了第一步、第二步之后发生线程切换，B 线程开始执行第一步、第二步、第三步（因为A 线程singleton 是还没有赋值的），所以为了保障这三个步骤不可中断，可以使用synchronized 在这段代码块上加锁。

**2.第一次检查singleton为空后为什么内部还进行第二次检查**

A 线程进行判空检查之后开始执行synchronized代码块时发生线程切换(线程切换可能发生在任何时候)，B 线程也进行判空检查，B线程检查 singleton == null 结果为true，也开始执行synchronized代码块，虽然synchronized 会让二个线程串行执行，如果synchronized代码块内部不进行二次判空检查，singleton 可能会初始化二次。

**3.volatile 除了内存可见性，还有别的作用吗？**

volatile 修饰的变量除了可见性，还能防止指令重排序。

**指令重排序** 是编译器和处理器为了优化程序执行的性能而对指令序列进行重排的一种手段。现象就是CPU 执行指令的顺序可能和程序代码的顺序不一致，例如 `a = 1; b = 2;` 可能 CPU 先执行`b=2;` 后执行`a=1;`

`singleton = new Singleton();` 由三步操作组合而成，如果不使用volatile 修饰，可能发生指令重排序。步骤3 在步骤2 之前执行，singleton 引用的是还没有被初始化的内存空间，别的线程调用单例的方法就会引发未被初始化的错误。