# 【Java容器】WeakHashMap

## 预备知识

Java提供了四种强度不同的引用类型；

**强引用：**

被强引用关联的对象不会被回收。

使用new一个新对象的方式来创建强引用。

```java
Object obj = new Object()
```

**软引用：**

被软引用关联的对象只有在内存不够的情况下才会被回收。

使用SoftReference类来创建软引用。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

**弱引用**

被弱引用关联的对象一定会被回收，也就是说它只能存活到下一层垃圾回收发生之前。

使用WeakReference类来创建弱引用。

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```

**虚引用**

又称为幽灵引用或者幻影引用，一个对象是否有虚引用的存在，不会对其生存时间造成影响，也无法通过虚引用得到一个对象。

为一个对象设置虚引用的唯一目的是能在这个对象被回收时收到一个系统通知。

使用 PhantomReference 来创建虚引用。

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj, null);
obj = null;
```





## 一、WeakHashMap的存储结构

WeakHashMap的Entry继承自WeakReference，被WeakReference关联的对象在下一次垃圾回收时会被回收。

WeakHashMap主要实现缓存，通过使用WeakHashMap来引用缓存对象，由JVM对这部分缓存进行回收。

```
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>
```

## 二、ConcurrentCache

Tomcat中的ConcurrentCache使用了WeakHashMap来实现缓存功能。

ConcurrentCache采取的是分代缓存：

- 经常使用的对象放入eden中，eden中使用ConcurrentHashMap实现，不用担心被回收(伊甸园)；
- 不常用的对象放入longterm，longterm中使用WeakHashMap实现，这些老对象会被垃圾收集器回收；
- 当调用get()方法时，会先从eden区获取，如果没有找到的话再到longterm获取，当从longterm获取到就把对象放入eden中，从而保证经常被访问的节点不容易被回收；
- 当调用put()方法时，如果eden的大小超过了size，那么就将eden中的所有对象都放入longterm中，利用虚拟机回收掉一部分不经常使用的对象；

```java
public final class ConcurrentCache<K, V> {

    private final int size;

    private final Map<K, V> eden;

    private final Map<K, V> longterm;

    public ConcurrentCache(int size) {
        this.size = size;
        this.eden = new ConcurrentHashMap<>(size);
        this.longterm = new WeakHashMap<>(size);
    }

    public V get(K k) {
        V v = this.eden.get(k);
        if (v == null) {
            v = this.longterm.get(k);
            if (v != null)
                this.eden.put(k, v);
        }
        return v;
    }

    public void put(K k, V v) {
        if (this.eden.size() >= size) {
            this.longterm.putAll(this.eden);
            this.eden.clear();
        }
        this.eden.put(k, v);
    }
}
```





