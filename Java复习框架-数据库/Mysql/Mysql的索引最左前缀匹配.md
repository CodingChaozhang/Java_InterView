# 数据库Mysql-索引的最左前缀匹配原则

**最左前缀匹配原则：**最左优先，以最左边的为起点任何连续的索引都能匹配上。同时如果范围查询(>、<、between、like)就会停止匹配。

## 一、例子来理解最左前缀匹配原则

前一篇文中，我们已经了解到Mysql数据库的索引的底层存储是一棵B+树，那么联合索引的底层也还是一棵B+树。只不过联合索引的键值对数量不是一个，而是多个。

假如：**构建一个(a,b)的联合索引，那么它在数据库底层的索引树是下列这样的：**

![image-20210312131335390](E:\笔记\JAVA\Java复习框架-数据库\Mysql\temp\1-1.png)

可以看到a的值是有顺序的，1，1，2，2，3，3，而b的值是没有顺序的1，2，1，4，1，2。所以b = 2这种查询条件没有办法利用索引，因为联合索引首先是按a排序的，b是无序的。

同时我们还可以发现**在a值相等的情况下，b值又是按顺序排列的，但是这种顺序是相对的。所以最左匹配原则遇上范围查询就会停止**，剩下的字段都无法使用索引。例如a = 1 and b = 2 a,b字段都可以使用索引，因为在a值确定的情况下b是相对有序的，而a>1and b=2，a字段可以匹配上索引，但b值不可以，因为a的值是一个范围，在这个范围中b是无序的。


## 二、最左前缀匹配原则适用场景

假如建立联合索引(a,b,c)

### 2.1 全值匹配查询时

```sql
select * from table_name where a = '1' and b = '2' and c = '3' 
select * from table_name where b = '2' and a = '1' and c = '3' 
select * from table_name where c = '3' and b = '2' and a = '1' 
```

用到了索引

where子句几个搜索条件顺序调换不影响查询结果，因为Mysql中有查询优化器，会自动优化查询顺序 

### 2.2 匹配左边的列时

```sql
select * from table_name where a = '1' 
select * from table_name where a = '1' and b = '2'  
select * from table_name where a = '1' and b = '2' and c = '3'
```

都从最左边开始连续匹配，用到了索引



```sql
select * from table_name where  b = '2' 
select * from table_name where  c = '3'
select * from table_name where  b = '1' and c = '3' 
```

这些没有从最左边开始，最后查询没有用到索引，用的是全表扫描 



```sql
select * from table_name where a = '1' and c = '3' 
```

如果不连续时，只用到了a列的索引，b列和c列都没有用到 



### 2.3 匹配列前缀

如果列是字符型的话它的比较规则是先比较字符串的第一个字符，第一个字符小的哪个字符串就比较小，如果两个字符串第一个字符相通，那就再比较第二个字符，第二个字符比较小的那个字符串就比较小，依次类推，比较字符串。

如果a是字符类型，那么前缀匹配用的是索引，后缀和中缀只能全表扫描了

```sql
select * from table_name where a like 'As%'; //前缀都是排好序的，走索引查询
select * from table_name where  a like '%As'//全表查询
select * from table_name where  a like '%As%'//全表查询
```

### 4.匹配范围值

```sql
select * from table_name where  a > 1 and a < 3
```

可以对最左边的列进行范围查询

```sql
select * from table_name where  a > 1 and a < 3 and b > 1;
```

多个列同时进行范围查找时，只有对索引最左边的那个列进行范围查找才用到B+树索引，也就是只有a用到索引，在1<a<3的范围内b是无序的，不能用索引，找到1<a<3的记录后，只能根据条件 b > 1继续逐条过滤

### 5. 精准匹配某一列并范围匹配另外一列

如果左边的列是精确查找的，右边的列可以进行范围查找

```sql
select * from table_name where  a = 1 and b > 3;
```

 a=1的情况下b是有序的，进行范围查找走的是联合索引

### 6.排序

一般情况下，我们只能把记录加载到内存中，再用一些排序算法，比如快速排序，归并排序等在内存中对这些记录进行排序，有时候查询的结果集太大不能在内存中进行排序的话，还可能暂时借助磁盘空间存放中间结果，排序操作完成后再把排好序的结果返回客户端。Mysql中把这种再内存中或磁盘上进行排序的方式统称为文件排序。文件排序非常慢，但如果order子句用到了索引列，就有可能省去文件排序的步骤

```sql
select * from table_name order by a,b,c limit 10;
```

因为b+树索引本身就是按照上述规则排序的，所以可以直接从索引中提取数据，然后进行回表操作取出该索引中不包含的列就好了

order by的子句后面的顺序也必须按照索引列的顺序给出，比如

```sql
select * from table_name order by b,c,a limit 10;
```

 这种颠倒顺序的没有用到索引

```sql
select * from table_name order by a limit 10;
select * from table_name order by a,b limit 10;
```


这种用到部分索引

```sql
select * from table_name where a =1 order by b,c limit 10;
```

联合索引左边列为常量，后边的列排序可以用到索引

