# MySQL学习笔记 

本章节内容主要是针对InnoDB存储引擎进行介绍，其他存储引擎，例如MyISAM等不在此章节讨论范围内。

## ONLINE DDL 操作
online DDL 特性支持就地的更改表和并发DML操作。该特性的优点包括:
- 在频繁交互的生产环境中，控制表在几分钟或者几个小时内不可用是不现实的，online DDL很好的解决了该问题；
- 在DDL操作过程中，本特性能控制 LOCK 子句在 DDL 操作期间调整性能和并发之间平衡的能力。请参见 LOCK 子句；
- 与基于table-copy方式相比，需要更少的磁盘空间和磁盘I/O

通常情况下，不需要执行任何特殊操作来启用online DDL。默认情况下，MySQL 在允许的情况下就地执行操作，尽可能少地对表进行锁定。
通过 `alter table` 语句的 `ALGORITHM` 和 `LOCK` 子句控制online DDL 操作。这些子句放在语句的末尾，用逗号与表和列规范分开。例如:
```sql
ALTER TABLE tbl_name ADD PRIMARY KEY (column), ALGORITHM=INPLACE, LOCK=NONE;
```
`LOCK` 子句对于微调对表的并发访问程度非常有用。ALGORITHM 子句主要用于性能比较，并且在遇到任何问题时作为回退到较早的表复制行为。例如:







## `INFORMATION_SCHEMA` 数据库 

### `INNODB_TRX` 表
`INNODB_TRX` 表提供了关于当前在 INNODB 引擎内部执行的每个事务的信息，包括事务是否正在等待锁定、事务何时开始以及事务执行的 SQL 语句(如果有的话)。

示例：

```sql
$mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX\G
*************************** 1. row ***************************
                    trx_id: 1510
                 trx_state: RUNNING
               trx_started: 2021-11-19 13:24:40
     trx_requested_lock_id: NULL
          trx_wait_started: NULL
                trx_weight: 586739
       trx_mysql_thread_id: 2
                 trx_query: DELETE FROM employees.salaries WHERE salary > 65000
       trx_operation_state: updating or deleting
         trx_tables_in_use: 1
         trx_tables_locked: 1
          trx_lock_structs: 3003
     trx_lock_memory_bytes: 450768
           trx_rows_locked: 1407513
         trx_rows_modified: 583736
   trx_concurrency_tickets: 0
       trx_isolation_level: REPEATABLE READ
         trx_unique_checks: 1
    trx_foreign_key_checks: 1
trx_last_foreign_key_error: NULL
 trx_adaptive_hash_latched: 0
 trx_adaptive_hash_timeout: 10000
          trx_is_read_only: 0
trx_autocommit_non_locking: 0
```


- `TRX_ID`

  事物ID。这些事务不是为只读和非锁定的事务创建的。详情请参阅第8.5.3节“优化 InnoDB 只读事务”。

- `TRX_WEIGHT`

  事物权重，反映事物影响的行数（比如：修改的行数和事物锁定的行数）。为了解决死锁，InnoDB 选择权重最小的事务作为"牺牲品"进行回滚。无论更改和锁定的行数多少，已更改非事务性表的事务都被认为比其他事务更重。 

- `TRX_STATE`

  事物执行状态，`RUNNING`, `LOCK WAIT`, `ROLLING BACK`, 和 `COMMITTING` 状态。

- `TRX_STARTED`

  事物开始时间

- `TRX_REQUESTED_LOCK_ID`

  当前事务正在等待的事务锁ID，如果 `TRX_STATE` 是 `LOCK WAIT`或者`NUll`。若要获取关于锁的详细信息，请将此列与`INNODB_LOCKS`表的`LOCK_ID`列联接起来。

- `TRX_WAIT_STARTED`

  事务开始等待锁的时间，如果 `TRX_STATE` 是 `LOCK WAIT`或者`NUll`。

- `TRX_MYSQL_THREAD_ID`

  mysql线程ID， 如果想获取更多详细信息，可以结合`INFORMATION_SCHEMA.PROCESSLIST`表的`ID`列查看。

- `TRX_QUERY`

  当前事务正在执行的SQL语句

- `TRX_OPERATION_STATE`
  
  当前事务操作

- `TRX_TABLES_IN_USE`
  
  处理此事务的当前 SQL 语句时使用的 InnoDB 表的数量。

- `TRX_TABLES_LOCKED`
  
  当前 SQL 语句上有行锁的 InnoDB 表的数量。(因为这些是行锁，而不是表锁，所以通常仍然可以由多个事务读取和写入表，尽管有些行被锁定。)

- `TRX_LOCK_STRUCTS`
  
  事务保留的锁的数量。

- `TRX_LOCK_MEMORY_BYTES`
  
  内存中此事务的锁结构占用的总大小。

- `TRX_ROWS_LOCKED`
  
  此事务锁定的大约行数。该值可能包括物理上存在但事务不可见的已删除标记的行。虽然列提到了“表”，但它并不是字面上的表锁，而是事务中包含一个或多个 InnoDB 行锁的表的数量。

- `TRX_ROWS_MODIFIED`

  此事务中已修改和插入的行数。

- `TRX_CONCURRENCY_TICKETS`
  
  一个值，指示当前事务在交换出去之前可以做多少工作，由 `innodb_concurrency_tickets` 系统变量指定。

- `TRX_ISOLATION_LEVEL`

  当前事务的隔离级别。

- `TRX_UNIQUE_CHECKS`
 
  当前事务是否打开或关闭唯一检查。例如，它们可能在大容量数据加载期间被关闭。

- `TRX_FOREIGN_KEY_CHECKS`
  
  当前事务是否打开或关闭外键检查。例如，它们可能在大容量数据加载期间被关闭。

- `TRX_LAST_FOREIGN_KEY_ERROR`
  
  最后一个外键错误(如果有的话)的详细错误消息; 否则为 NULL。

- `TRX_ADAPTIVE_HASH_LATCHED`
  
  当前事务是否锁定自适应哈希索引。当对自适应哈希索引搜索系统进行分区时，单个事务不锁定整个自适应哈希索引。自适应哈希索引分区由 innodb _ Adaptive _ hash _ index _ parts 控制，默认设置为8。

- `TRX_ADAPTIVE_HASH_TIMEOUT`
  
  是否立即放弃自适应哈希索引的搜索锁，或者保留对Mysql的调用。当没有自适应哈希索引争用时，该值保持为零，并且语句在完成之前保留搜索锁。在锁竞争期间，它将计数为零，语句在每次行查找到后立即释放锁。对自适应哈希索引搜索系统进行分区(由 `innodb_adaptive_hash_index_parts` 控制)时，其值保持为0。

- `TRX_IS_READ_ONLY`
  
  该值为 1 代表事务只读。

- `TRX_AUTOCOMMIT_NON_LOCKING`
  
  值为1表示该事务是一个 `SELECT` 语句，该语句不使用 `FOR UPDATE `或 `LOCK IN SHARED MODE` 子句，并且在启用自动提交的情况下执行，因此该事务只包含这一条语句。当这个列和`TRX_IS_READ_ONLY`的值都为1时，InnoDB 优化事务以减少与更改表数据的事务相关的开销。


> **注意事项**
>  - 使用此表可帮助诊断在重并发负载期间发生的性能问题。其内容更新如第14.16.2.3节“ InnoDB 事务和锁定信息的持久性和一致性”所述。
>  - 必须具有 PROCESS 特权才能查询此表。
>  - 使用 `information_schema.COLUMNS` 表或 `SHOW COLUMNS` 语句查看有关此表列的其他信息，包括数据类型和默认值。





## 附录：缓冲池 vs 数据页 vs 表空间 vs 段 vs 区 vs 数据行

### [缓冲池（Buffer pool）](https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool.html)

**缓冲池**是主存中的一个区域，InnoDB 在访问时缓存表和索引数据。缓冲池允许直接从内存访问频繁使用的数据，加快处理速度。在数据库专用服务器上，高达80% 的物理内存通常分配给缓冲池。

![](./innodb-buffer-pool-list.png)

当 InnoDB 将一个页面读入缓冲池时，它首先将其插入到中点（Midpoint Insertion）(旧子列表的头部)。比如用户发起的操作(如 SQL 查询)所必需的，或者 InnoDB 自动执行的预读操作的一部分。随着数据库的运行，频繁访问的数据由缓冲池中old sublist移动到列表的head部，再移动到new sublist中。新旧子列表（new-old sublist）中的页面都会随着其他页面的更新而老化。旧子列表(old sublist)中的页面也会随着页面插入到中点而变老。最终，一个未使用的页面到达旧子列表的尾部并被驱逐。


### 表空间 vs 段 vs 区 vs 页 vs 数据行

表空间、段、区、页、数据行均是占用的磁盘空间，如下图所示，说明了表空间 vs 段 vs 区 vs 页 之间的关系。

![表空间、段、区、页、数据行](./4183291211-3b3d40e37b3967ab_fix732.png)


**数据行**：从上图可见，数据行是构成数据页必不可少的一种数据结构。数据库中的每行数据按照某一种行格式存储某一种文件格式中，同时以该种文件格式存储在磁盘中。

**[行格式](https://dev.mysql.com/doc/refman/5.7/en/innodb-row-format.html) vs [文件格式](https://dev.mysql.com/doc/refman/5.7/en/innodb-file-format.html)**

MySQL 5.1 以后的版本，支持2种文件格式`Antelope`（羚羊）、`Barracuda`（梭鱼）。4种行格式：`REDUNDANT`、`COMPACT`、`DYNAMIC`、`COMPRESSED`。

一个表可以指定一个行是以什么样的格式进行存储，例如如下代码：

```sql
CREATE TABLE customer (
name VARCHAR(10) NOT NULL, address VARCHAR(20),
gender CHAR(1),
job VARCHAR(30),
school VARCHAR(50)
) ROW_FORMAT=COMPACT;
```

每种文件格式又支持一种或者多种行格式，如下表所示：

|行格式	|紧凑的存储特性	|增强的可变长度列存储	|大索引键前缀支持|	压缩支持|	支持的表空间类型|	所需文件格式|
|-----|-----|-----|-----|-----|-----|-----|
|REDUNDANT|	不|	不|	不|	不|	系统，每表文件，一般|	羚羊或梭鱼|
|COMPACT|	是的|	不|	不	|不|	系统，每表文件，一般	|羚羊或梭鱼|
|DYNAMIC|	是的|	是的|	是的|	不|	系统，每表文件，一般|	梭鱼|
|COMPRESSED|	是的|	是的|	是的|	是的|	每表文件，一般|	梭鱼|

参考文章: [《数据行结构和行溢出机制》](https://alsritter.icu/posts/d2ad62f9/)

**数据页（page）**：数据库从磁盘中读取数据的最小单位是数据页，但数据页里不是一行一行的数据，其实一个数据页包含了下面的部分：文件头，数据页头，最大最小记录，多个数据行和空闲区域，最后是数据页目录和文件尾部。数据页中包含的数据行越多，在查询和索引时速率越快，同时占用的buffer pool 和 I/O 资源越少。

数据页的写入原理：我们前面提到，数据库从磁盘读取的最小单位是数据页，（假设按聚簇索引顺序写入）将目标页加载到（[InnoDB 数据页解析](http://mysql.taobao.org/monthly/2018/04/03/)）buffer pool中。当遇到insert语句时，直接将数据写入到buffer pool加载后的数据页中(此时，并没有刷盘持久化，为了避免server宕机导致的buffer pool 数据丢失，数据库提供了redo log和binlog机制)，写入到数据页的数据根据聚集索引排列的，根据聚簇索引，在满足一个数据页存储的容量（16K）情况下，相邻的行数据存储在同一数据页中。针对页分裂，本部分不做介绍（当行的主键值要求必须将这一行插入到某个已满的页中时，存储引擎会将该页分裂成两个页面来容纳该行，这就是一次分裂操作。页分裂会导致表占用更多的磁盘空间）。


**数据区(extent)**：一个数据区对应64个数据页，一个数据页是16kb，那么一个数据区是1MB，256个数据区被划分为一组，对于表空间而言，他的第一组数据区的第一个数据区的前3个数据页，都是固定的，里面存放了一些描述性的数据。比如FSP_HDR这个数据页，他里面就存放了表空间和这一组数据区的一些属性。IBUF_BITMAP 数据页，存放的就是insert buffer的信息，INODE 数据页存放的也是特殊信息。



**数据段(segment)**：段(Segment)分为索引段，数据段，回滚段等。其中索引段就是非叶子结点部分，而数据段就是叶子结点部分，回滚段用于数据的回滚和多版本控制。一个段包含256个区(256M大小)。​ 一个段包含256个区。

**表空间(tablespace)**：为磁盘空间一组逻辑上的空间区域。 当我们创建一个表之后，在磁盘上会有对应的表名称.ibd的磁盘文件。表空间的磁盘文件里面有很多的数据页，一个数据页最多16kb，因为不可能一个数据页一个磁盘文件，所以数据区的概念引入了。每个表空间就是对应了磁盘上的数据文件，在表空间里有很多组数据区，一组数据区是256个数据区， 每个数据区包含了64个数据页，是1mb   

