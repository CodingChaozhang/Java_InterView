# Mysql如何恢复数据？如何进行主动复制？Binlog日志到底是什么？

## 1.Binlog日志简介

在MySQL中一般有以下几种日志：

| 日志类型                 | 写入日志的信息                                               |
| ------------------------ | ------------------------------------------------------------ |
| **错误日志**             | 记录在启动，运行或停止mysqld时遇到的问题                     |
| **通用查询日志**         | 记录建立的客户端连接和执行的语句                             |
| **二进制日志（Binlog）** | **记录更改数据的语句**                                       |
| 中继日志                 | 从复制主服务器接收的数据更改                                 |
| **慢查询日志**           | 记录所有执行时间超过`long_query_time`秒的所有查询或不使用索引的查询 |
| DDL日志(元数据日志)      | 元数据操作由DDL语句执行                                      |

本次的二进制日志Binlog可以说是Mysql最重要的日志，其记录了所有的`DDL`和`DML`语句(除了数据查询语句select，show等)，以事件形式记录，还包含语句所执行的消耗时间，Mysql的二进制日志是事务安全型的。binlog的主要目的是**复制和恢复**。

## 2 Binlog日志的两个最重要的使用场景

- **Mysql主从复制：**Mysql Replication在Master端开启binlog，Master把它的二进制日志传递给slaves来达到master-slave数据一致的目的。
- **数据恢复：**通过使用mysqlbinlog工具来恢复数据。

## 3 启用Binlog日志

一般来说开启binlog日志大概会有1%的性能损耗。

启用binlog，通过配置 `/etc/my.cnf` 或 `/etc/mysql/mysql.conf.d/mysqld.cnf` 配置文件的 `log-bin` 选项：

在配置文件中加入 `log-bin` 配置，表示启用binlog，如果没有给定值，写成 `log-bin=`，则默认名称为主机名。（注：名称若带有小数点，则只取第一个小数点前的部分作为名称）

```
[mysqld]
log-bin=my-binlog-name
12
```

也可以通过 `SET SQL_LOG_BIN=1` 命令来启用 binlog，通过 `SET SQL_LOG_BIN=0` 命令停用 binlog。启用 binlog 之后须重启MySQL才能生效。

## 4.Binlog日志的写入时机

对支持事务的引擎如InnoDB而言，必须要提交了事务才会记录binlog。binlog 什么时候**刷新到磁盘**跟参数 `sync_binlog` 相关。

- 如果设置为0，则表示**MySQL不控制binlog的刷新，由文件系统去控制它缓存的刷新**；
- 如果设置为不为0的值，则表示每 `sync_binlog` 次事务，**MySQL调用文件系统的刷新操作刷新binlog到磁盘中**。
- 设为1是最安全的，在系统故障时最多丢失一个事务的更新，但是会对性能有所影响。

如果 `sync_binlog=0` 或 `sync_binlog大于1`，**当发生电源故障或操作系统崩溃时，可能有一部分已提交但其binlog未被同步到磁盘的事务会被丢失，恢复程序将无法恢复这部分事务。**

在MySQL 5.7.7之前，默认值 sync_binlog 是0，MySQL 5.7.7和更高版本使用默认值1，这是最安全的选择。一般情况下会设置为100或者0，牺牲一定的一致性来获取更好的性能。

## 5.常用的BinLog操作命令

```java
# 是否启用binlog日志
show variables like 'log_bin';

# 查看详细的日志配置信息
show global variables like '%log%';

# mysql数据存储目录
show variables like '%dir%';

# 查看binlog的目录
show global variables like "%log_bin%";

# 查看当前服务器使用的biglog文件及大小
show binary logs;

# 查看主服务器使用的biglog文件及大小

# 查看最新一个binlog日志文件名称和Position
show master status;


# 事件查询命令
# IN 'log_name' ：指定要查询的binlog文件名(不指定就是第一个binlog文件)
# FROM pos ：指定从哪个pos起始点开始查起(不指定就是从整个文件首个pos点开始算)
# LIMIT [offset,] ：偏移量(不指定就是0)
# row_count ：查询总条数(不指定就是所有行)
show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];

# 查看 binlog 内容
show binlog events;

# 查看具体一个binlog文件的内容 （in 后面为binlog的文件名）
show binlog events in 'master.000003';

# 设置binlog文件保存事件，过期删除，单位天
set global expire_log_days=3; 

# 删除当前的binlog文件
reset master; 

# 删除slave的中继日志
reset slave;

# 删除指定日期前的日志索引中binlog日志文件
purge master logs before '2019-03-09 14:00:00';

# 删除指定日志文件
purge master logs to 'master.000003';

```

## 6 Binlog文件

binlog日志包括两类文件:

- **二进制日志索引文件**（文件名后缀为.index）用于记录所有有效的的二进制文件
- **二进制日志文件（**文件名后缀为.00000*）记录数据库所有的DDL和DML语句事件

binlog是一个二进制文件集合，每个binlog文件以一个4字节的魔数开头，接着是一组Events:

- 魔数：0xfe62696e对应的是0xfebin；
- Event：每个Event包含header和data两个部分；header提供了Event的创建时间，哪个服务器等信息，data部分提供的是针对该Event的具体信息，如具体数据的修改；
- 第一个Event用于描述binlog文件的格式版本，这个格式就是event写入binlog文件的格式；
- 其余的Event按照第一个Event的格式版本写入；
- 最后一个Event用于说明下一个binlog文件；
- binlog的索引文件是一个文本文件，其中内容为当前的binlog文件列表

当遇到以下3种情况时，MySQL会重新生成一个新的日志文件，文件序号递增：

- MySQL服务器停止或重启时
- 使用 `flush logs` 命令；
- 当 binlog 文件大小超过 `max_binlog_size` 变量的值时；

> `max_binlog_size` 的最小值是4096字节，最大值和默认值是 1GB (1073741824字节)。事务被写入到binlog的一个块中，所以它不会在几个二进制日志之间被拆分。因此，如果你有很大的事务，为了保证事务的完整性，不可能做切换日志的动作，只能将该事务的日志都记录到当前日志文件中，直到事务结束，你可能会看到binlog文件大于 max_binlog_size 的情况。

## 7 Mysql的binlog有几种记录格式？分别有什么区别？

记录在二进制日志中的事件的格式取决于二进制记录格式。支持三种格式类型：

- STATEMENT：基于SQL语句的复制（statement-based replication, SBR）
- ROW：基于行的复制（row-based replication, RBR）
- MIXED：混合模式复制（mixed-based replication, MBR）

在 `MySQL 5.7.7` 之前，默认的格式是 `STATEMENT`，在 `MySQL 5.7.7` 及更高版本中，默认值是 `ROW`。日志格式通过 `binlog-format` 指定，如 `binlog-format=STATEMENT`、`binlog-format=ROW`、`binlog-format=MIXED`。

- statement模式下，每一条会修改数据的sql都会记录在binlog中。不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。由于sql的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。
- row级别下，不记录sql语句上下文相关信息，仅保存哪条记录被修改。记录单元为每一行的改动，基本是可以全部记下来但是由于很多操作，会导致大量行的改动(比如alter table)，因此这种模式的文件保存的信息太多，日志量太大。
- mixed，一种折中的方案，普通操作使用statement记录，当无法使用statement的时候使用row。

## 8.Mysql的主从复制

| 复制方式   | 操作                                                         |
| ---------- | ------------------------------------------------------------ |
| 异步复制   | 默认异步复制，容易造成主库数据和从库不一致,一**个数据库为Master**, **一个数据库为slave**,通过Binlog日志,**slave两个线程，一个线程去读master binlog日志**，写到自己的中继日志；**一个线程解析日志，执行sql,**  **master启动一个线程,给slave传递binlog日志** |
| 半同步复制 | 只有把master发送的binlog日志写到slave的中继日志，这时主库,才返回操作完成的反馈，性能有一定降低 |
| 并行操作   | **slave 多个线程去请求binlog日志**                           |

### 8.1 Mysql主从复制的原理

- **在主库上把数据更高记录到二进制日志**
- **从库将主库的日志复制到自己的中继日志**
- **从库读取中继日志的事件，将其重放到从库数据中**



### 8.2 Mysql主从复制解决的问题

- **数据分布： **随意开始或停止复制，并在不同地理位置分布数据备份；
- **负载均衡**： 降低单个服务器的压力；
- **高可用和故障切换：** 帮助应用程序避免单点失败；
- **升级测试： **可以用更高版本的Mysql作为从库



### 8.3 Mysql主从复制的作用

- **主数据库出现问题，可以切换到从数据库；**
- **数据库层面的读写分离；**
- **在数据库上进行日常备份；**