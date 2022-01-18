# InnoDB可重复读隔离级别下如何避免幻读

## 1.当前读和快照读

- **快照读:**最简单的select操作,属于快照读,不加锁

  ```sql
  select * from table where id = 1;
  ```

- **当前读:**特殊的读与增删改操作,属于当前读,会读取数据库原本的数据,加锁

```sql
select * from table where xxx lock in share mode; (共享锁)
select * from table where xxx for update;
insert into table values()
update table set xxx where xxx
delete form table where xxx
```

## 2. 四种隔离级别(由低到高)

Read Uncommitted读未提交：**可以看到其它事务未提交的内容；**

Read Commited读已提交：**可以看到其它事务已提交的内容；**

Repeatable Read可重复读：**事务开始时和事务结束时读到的数据完全相同；**

Serializable串行：**事务必须逐步执行，后来的会排队。**

## 3.隔离级别下的问题

前提条件：同时开启A和B两个事务；

**脏读：**

事务查询id为1的数据，num字段为1
B事务将id为1的数据num更新为2，但未commit
事务查询id为1的数据，num字段变为2
B回滚，则A读到的数据为脏数据

**不可重复读：**

事务查询id为1的数据，num字段为1
B事务将id为1的数据num更新为2，commit
事务查询id为1的数据，num字段变为2
A事务前后两次读到的同一条记录num字段不同，不可重复读

**幻读：**

A事务查询id >=1 and id <=3的数据，得到id=1和id=3两条数据（注意：没有id=2的数据）
B事务插入id=2的数据
A事务再查询id >=1 and id <=3的数据，发现多了一条id=2的数据，即两次查询数据的行数不同

**各种隔离模式下会出现的问题**

读未提交：脏读、不可重复读、幻读；

读已提交：不可重复读、幻读；

可重复读：幻读；

## 4.next-key

### 4.1 间隙锁(gap-key)

**在一个事务中select for update查询某条记录，会锁定该条记录的前后空行**，什么叫空行呢，看个例子就知道了

id num
1 2
2 4
3 6
4 7
执行select * from table where num = 4 for update

insert into table ('', 3)和insert into table ('', 5)都会被阻塞，即4前后的空行都无法插入

### 4.2 next-key

**由于gay-key只能锁定记录之间的间隙，但是我们上面查询num=4的行也不能被更改，索引该行也会被加行锁，此时这种既能加行锁，又能加间隙锁的，称之为next-key**。



## 5. MVCC

事务A：查询num>=2 and num<=4的记录，但是不加for update！不加for update！不加for update！
事务B：insert into table ('', 3)，不会被阻塞，不会被阻塞，不会被阻塞！然后commit
事务A：查询num>=2 and num<=4的记录，和之前查询相同
事务A：提交
事务A：查询num>=2 and num<=4的记录，可以查询到num=3的记录，此时返回3条结果
原理：假如事务A在开启的时候版本号为2，当更改或者插入数据后，该条记录的版本号+1，也就是说事务B插入num=3的记录的版本号为3，**事务A在提交前的所有select都只能查询到版本号<=2的记录，也就是说不会产生上面说的幻读。**

## 6.总结

**MVCC和next-key**

**其实，innodb采用next-key + MVCC去解决幻读问题的：**

**在查询加for update时，会用next-key解决幻读问题，新的insert和update会阻塞**
**在查询不加for update时，会用MVCC解决幻读问题，新的insert和update不会阻塞**

