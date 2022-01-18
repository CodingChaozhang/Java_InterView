# 【Java容器】数据结构-LRU算法

## 1.LRU是什么？

**LRU全称Least Recently Used**，也就是最近最少使用的意思，是一种内存管理算法，最早应用于Linux操作系统。

LRU算法基于一种假设：长期不被使用的数据，在未来被用到的几率也不大。因此，当数据所占内存达到一定阈值时，我们要移除掉最近最少被使用的数据。

LRU算法应用：可以在内存不够时，从哈希表移除一部分很少访问的用户。



LRU是什么？按照英文的直接原义就是Least Recently Used,最近最久未使用法，它是按照一个非常著名的计算机操作系统基础理论得来的：**最近使用的页面数据会在未来一段时期内仍然被使用,已经很久没有使用的页面很有可能在未来较长的一段时间内仍然不会被使用**。基于这个思想,会存在一种缓存淘汰机制，每次从内存中找到最久未使用的数据然后置换出来，从而存入新的数据！它的主要衡量指标是使用的时间，附加指标是使用的次数。在计算机中大量使用了这个机制，它的合理性在于优先筛选热点数据，所谓热点数据，就是最近最多使用的数据！因为，利用LRU我们可以解决很多实际开发中的问题，并且很符合业务场景。

## 2.LRU的需求

首先考虑这样的一个业务场景,小王在A公司上班，有一天产品提出了一个需求：“咱们系统的用户啊，每天活跃的就那么多，有太多的僵尸用户，根本不登录，你能不能考虑做一个筛选机制把这些用户刨出去，并且给活跃的用户做一个排名，我们可以设计出一些奖励活动，提升咱们的用户粘性，咱们只需要关注那些活跃的用户就行了“”。小王连忙点头，说可以啊，然而心里犯起嘀咕来了：这简单，按照常规思路，给用户添加一个最近活跃时间长度和登录次数，然后按照这两个数据计算他们的活跃度，最后直接排序就行了。嘿嘿，简直完美!不过！用户表字段已经很多了，又要加两个字段，然后还得遍历所有的数据排序？这样查询效率是不是会受影响啊？

## 3.LRU的实现方式

实现 LRUCache 类：

- LRUCache(int capacity) 以正整数作为容量 capacity 初始化 LRU 缓存；
- int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
- void put(int key, int value) 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

### 3.1 直接继承LinkedHashMap来实现

```java
public class Code_LRU {
	class LRUCache extends LinkedHashMap<Integer,Integer>{
		private int capacity;
		public LRUCache(int capacity) {
			super(capacity,0.75F,true);
			this.capacity = capacity;
		}
		
		public int get(int key) {
			return super.getOrDefault(key,-1);
		}
		public void put(int key,int value) {
			super.put(key, value);
		}
		
		// 重写淘汰策略
		protected boolean removeEldestEntry(Map.Entry<Integer, Integer> edlest) {
			return size()>capacity;
		}
	}
}

```

### 3.2 用双向链表+hashMap来实现

```java
// 解题思路：双向链表+hashmap来实现
	class LRUCache_2{
		// 双向链表
		class DLinkedNode{
			int key;
			int value;
			DLinkedNode prev;
			DLinkedNode next;
			public DLinkedNode() {
				
			}
			public DLinkedNode(int key,int value) {
				this.key = key;
				this.value = value;
			}
		}
		
		// hashmap
		private HashMap<Integer,DLinkedNode> cache = new HashMap<>();
		// 定义私有变量
		private int size;
		private int capacity;
		private DLinkedNode head,tail;
		
		public LRUCache_2(int capacity) {
			this.size = 0;
			this.capacity = capacity;
			// 生成伪头部和伪尾部结点
			head = new DLinkedNode();
			tail = new DLinkedNode();
			head.next = tail;
			tail.prev = head;
		}
		
		//获得值
		public int get(int key) {
			DLinkedNode node = cache.get(key);
			if(node==null) {
				return -1;
			}else {
				// 如果key存在，哈希表定位 结点移动到头部
				moveToHead(node);
				return node.value;
			}
		}
		
		// 放入值的操作
		public void put(int key,int value) {
			DLinkedNode node = cache.get(key);
			// 如果key不存在
			if(node==null) {
				// 新创建一个结点
				DLinkedNode newNode = new DLinkedNode(key,value);
				// 添加
				cache.put(key,newNode);
				// 添加到头部
				addToHead(newNode);
				++size;
				// 判断容量
				if(size>capacity) {
					// 称号出容量
					DLinkedNode tail = removeTail();
					// 删除hash表中对应的项
					cache.remove(tail.key);
					--size;
				}
				
			}else {
				node.value = value;
				// 移动
				moveToHead(node);
			}
		}
		
		// 添加结点的操作
		private void addToHead(DLinkedNode node) {
			node.prev = head;
			node.next = head.next;
			head.next.prev = node;
			head.next = node;
		}
		
		// 删除结点
		private void removeNode(DLinkedNode node) {
			node.prev.next = node.next;
			node.next.prev = node.prev;
		}
		
		// 移动到头结点 删结点 填充结点
		private void moveToHead(DLinkedNode node) {
			removeNode(node);
			addToHead(node);
		}
		// 删除尾部结点
		private DLinkedNode removeTail() {
			DLinkedNode res = tail.prev;
			removeNode(res);
			return res;
		}
	}

```

