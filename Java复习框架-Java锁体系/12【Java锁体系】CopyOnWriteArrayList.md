# 【Java锁体系】-CopyOnWriteArrayList

## 一、背景

说到CopyOnWriteArrayList就需要说下，ArrayList。ArrayList是非线程安全的，也就是说在多个线程下进行读写，会出现异常。

那么ArrayList既然是非线程安全的，那么我们使用一些机制把它变为安全的就可以了。比如说替换成Vector，再或者是Collections，可以将ArrayList包装成一个线程安全的类。

不过Vector和collections的包装，有一个很大的缺点，就是它们使用的都是独占锁，独占锁在同一时刻只能有一个线程获取，效率太低。于是CopyOnWriteArrayList出现了。

## 二、CopyOnWriteArrayList介绍

### (1)独占锁效率低：采用读写分离思想解决

读操作不加锁，所有线程都不会阻塞。写操作加锁，线程会被阻塞；

### (2)写线程获取到锁，其它线程包括读线程阻塞

但是这时候又出现了另外一个问题：写线程获取到锁之后，其它的读线程会陷入阻塞

### (3)复制思想

当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行copy，复制出一个新的容器，然后往新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。

但这时候会抛出一个新的问题，也就是数据不一致的问题。如果写线程还没来得及写回内存，其他线程就会读到脏数据。这是copyonwritelock的缺点。



其适用场景就是读多写少的情况下比较好。

## 三、CopyOnWriteArrayList的实现

### 1.get方法

```java
/**
     * {@inheritDoc}
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
	// get这个方法很简单，没有加锁，主要通过getarray来实现的。
    public E get(int index) {
        return get(getArray(), index);
    }
/**
     * Gets the array.  Non-private so as to also be accessible
     * from CopyOnWriteArraySet class.
     */
    final Object[] getArray() {
        return array;
    }

```

整个读的过程没有添加任何锁，就是普通的数组获取。

### 2.add方法

```java
/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

写操作添加了一个锁ReentrantLock，这个锁可以决定在什么时候加锁和释放更加灵活。保证了写操作线程安全。能够体现copy的思想是Arrays.copyOf(elements,len+1),也就是数组复制了一份，最后setArray回去了。

## 四、CopyOnWriteArrayList和ConcurrentReadWriteLock总结

**读写锁ConcurrentReadWriteLock**

读线程具有实时性，写线程会阻塞，**解决了数据不一致的问题**。但是读**写锁依然会出现读线程阻塞等待的情况。**

**CopyOnWriteArrayList**

读线程具有实时性，写线程会阻塞。**不能解决数据不一致的问题**。但是**CopyOnWriteArrayList不会出现读线程阻塞等待的情况。**

