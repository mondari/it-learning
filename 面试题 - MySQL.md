[TOC]

# 基础篇

## DQL、DML、DDL、DCL的概念与区别

- DQL：数据查询语言(Data Query Language)，主要是 SELECT 查询
- DML：数据操作语言(Data Manipulation Language)，主要有三种形式：
  - 插入：INSERT
  - 更新：UPDATE
  - 删除：DELETE
- DDL：数据定义语言(Data Definition Language)，主要是创建数据库的各种对象，比如表、视图、索引等
  - CREATE TABLE
  - CREATE VIEW
  - CREATE INDEX
- DCL：数据控制语言(Data Control Language)，主要是权限授予和回收、事务提交和回滚等
  - GRANT
  - REVOKE
  - COMMIT
  - ROLLBACK



参考：[DQL、DML、DDL、DCL的概念与区别](https://www.cnblogs.com/fan-yuan/p/7879353.html)

## SQL 的执行流程

要执行一条 SQL 语句，首先要连接数据库，而 MySQL 管理连接的就是连接器。

- 连接器：管理连接，验证登录，判断用户名和密码是否正确。连接成功会查询缓存，如果缓存命中，则直接返回结果。
- 分析器：对 SQL 语句进行词法分析、语法分析并生成解析树。
- 优化器：根据解析树生成执行计划，并选择合适的索引（给出 SQL 语句的最优的执行方案）
- 执行器：调用存储引擎接口执行 SQL 语句，在执行前会判断是否对相应的表有相应的权限

一般情况下不建议使用查询缓存，因为查询缓存的缓存失效非常频繁，只要对一个表进行更新操作，这个表上的所有查询缓存都会被清空。因此很可能我们很费劲地把结果存起来，还没使用就被一个更新操作给全清空了。对于更新压力大的数据库来说，查询缓存的命中率会非常低。而且 MySQL 8.0 版本直接将查询缓存的整块功能给删掉了，也就是说 8.0 开始彻底没有这个功能了。

![img](面试题 - MySQL.assets/CgotOV14ySKAMxohAAH2VHcAzkE612.png)



### 查询语句的执行流程



### 更新语句的执行流程



参考：

[拉勾教育 - 高性能MySQL实战 - 第01讲：MySQL体系结构与存储引擎](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=5#/detail/pc?id=48)

[极客时间 - MySQL 实战 45讲 - 01 | 基础架构：一条SQL查询语句是如何执行的？](https://time.geekbang.org/column/article/68319)



## MySQL 支持哪些存储引擎，它们的区别？

执行 `SHOW ENGINES;` 命令可以查看支持的存储引擎；

InnoDB 和 MyISAM 存储引擎的对比：

| 功能     | InnoDB | MyISAM       |
| -------- | ------ | ------------ |
| 事务     | 是     | 否           |
| 行级锁   | 是     | 只支持表级锁 |
| 外键     | 是     | 否           |
| 崩溃恢复 | 是     | 否           |


## 主从复制原理

主数据库启动二进制日志（Binary Log），记录数据变更操作；从数据库会监听主数据库的二进制日志，如果监测到变化，就会启动一个 IO 线程向主数据库发送读请求，主数据库会为每个从数据库的 IO 线程启动一个 Dump 线程发送二进制日志，从数据库的 IO 线程会将接收到的二进制日志保存到本地的中继日志（Relay Log）中，然后启动 SQL 线程读取中继日志进行回放操作。

总结：

- 主数据库会启动 Dump 线程，从数据库会启动 IO 线程和 SQL 线程
- Dump 线程负责发送二进制日志
- IO 线程负责读取和接收二进制日志并将其保存到中继日志中
- SQL 线程负责读取中继日志并回放操作。

主从复制后，可以实现读写分离，主库写，从库读，从库读负载均衡，降低数据库的读写压力。

参考：https://github.com/alibaba/canal



## InnoDB Cluster 与 MySQL NDB Cluster



# 索引篇

## MySQL 支持哪些索引？

MySQL 支持五种索引，分别是 PRIMARY，KEY，UNIQUE，FULLTEXT，SPATIAL，即主键索引，普通索引，唯一索引、全文索引和空间索引

参考：[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

## 索引是怎样提高查询效率？

使用索引可以避免全表扫描。索引就像一本书的目录一样，通过目录去查比一页一页地翻快。

## 索引的数据结构

MySQL 的索引结构支持哈希表、B+树和R树（空间索引）。

### 哈希表

适合等值查询，不适合范围查询

![img](面试题 - MySQL.assets\0c62b601afda86fe5d0fe57346ace957.png)

### 有序数组

适合等值查询和范围查询

使用二分查找法，算法复杂度是O(log(N))。但是插入性能低，所以适合存储静态数据。

![img](面试题 - MySQL.assets\bfc907a92f99cadf5493cf0afac9ca49.png)

### B树

树的查询效率高(算法复杂度 `O(log(n))` )，而且能够保证有序，并且树的高度比二叉树低，有利于减少磁盘IO。

B树的特征：

- 所有叶子节点具有相同的深度
- 中间节点保存指针和数据，叶子节点不保存指针，只保存数据。这里的数据在 MySQL 中指的的是索引和卫星数据

### B+树

- （最重要）叶子节点有指向下一个叶子结点的指针，形成一个有序链表，所以**范围查询**的效率比B树高
- 叶子节点包含了所有元素。
- 在 MySQL 中，中间节点的元素都同时存在其子节点中，并且在子节点中是最大或最小的元素。B树没有这种冗余设计。
- 在 MySQL 中，中间节点只保存索引(和指针)，不保存数据，**只有叶子节点才保存数据（InnoDB 保存的是整行数据，而 MyISAM 保存的是卫星数据）**。由于中间节点不保存数据，所以能够容纳更多的索引



参考：

1. [极客时间 - MySQL 实战 45讲 - 04 | 深入浅出索引（上）](https://time.geekbang.org/column/article/69236)
2. [漫画：什么是B-树？](https://mp.weixin.qq.com/s/rDCEFzoKHIjyHfI_bsz5Rw)
3. [漫画：什么是B+树？](https://mp.weixin.qq.com/s/jRZMMONW3QP43dsDKIV9VQ)





## 聚集索引和辅助索引

- 聚集索引|聚簇索引(clustered index)：也叫主键索引，其叶子节点存的是整行数据。聚集指的是索引和数据聚集在一起存放。
- 辅助索引：也叫二级索引，其叶子节点存的是主键的值。查询完辅助索引后需要根据主键值再到聚集索引中查到整行数据，这个过程叫做**回表**。就是说，**基于辅助索引的查询需要多扫描一棵索引树**，因此，我们应该尽量使用主键去查询。

如图所示(这是B树，而不是B+树)，其中ID是主键索引字段，k是普通索引字段：

![img](面试题 - MySQL.assets/dcda101051f28502bd5c4402b292e38d.png)

基于聚簇索引和辅助索引的查询有什么区别？

- 如果语句是 select * from T where ID=500，即主键查询方式，则只需要搜索 ID 这棵 B+ 树；

- 如果语句是 select * from T where k=5，即普通索引查询方式，则需要先搜索 k 索引树，得到 ID 的值为 500，再到 ID 索引树搜索一次。

参考：[极客时间 - MySQL 实战 45讲 - 04 | 深入浅出索引（上）](https://time.geekbang.org/column/article/69236)

## 什么时候用(整型)自增主键？什么时候可以不用？

这涉及到索引的维护以及索引的大小。

**索引的维护**：往索引插入新值时，需要对索引进行必要的维护，以保持索引的有序性。

**自增主键**有利于减少索引的维护，因为自增主键本来就是有序的，只需要往索引后面插入新值即可。

**索引的大小**：**整型作为主键**有利于降低索引的大小。int 只要 4 个字节，bigint 则是 8 个字节，而 varchar 每个字符占用 1 个字节。所以使用整型的自增主键有利于减少普通索引的叶子节点占用的空间，从而降低普通索引的大小。

**什么时候可以不用自增主键**？

只有一个索引的场景。由于没有其他索引，所以也就不用考虑其他索引的叶子节点大小的问题。



参考：[极客时间 - MySQL 实战 45讲 - 04 | 深入浅出索引（上）](https://time.geekbang.org/column/article/69236)

## 联合索引

### *如何存储？



### 最左匹配

使用了 B+ 树的 MySQL 存储引擎，比如 InnoDB，在每次查询复合字段时是从左往右匹配数据的，因此在创建联合索引的时候需要注意索引创建的顺序。例如，我们创建了一个联合索引是 idx(name,age,sex)，那么当我们使用，姓名+年龄+性别、姓名+年龄、姓名等这种最左前缀查询条件时，就会触发联合索引进行查询；然而如果非最左匹配的查询条件，例如，性别+姓名这种查询条件就不会触发联合索引。

当然，当我们已经有了（name,age）这个联合索引之后，一般情况下就不需要在 name 字段单独创建索引了，这样就可以少维护一个索引。

### 覆盖索引|索引覆盖

辅助索引的叶子节点存的是主键值，如果根据辅助索引去查询主键值，则不需要再回表去查询聚集索引了，因为辅助索引已经“**覆盖了**”我们的查询需求，我们称这种查询过程为**索引覆盖**，也叫**覆盖索引**。

比如下面 SQL 语句根据名称去查询 ID，假设名称 name 上已经建立了索引，则通过 name 去查 id 不需要再回表去查询聚集索引了

```sql
select id from user where name = 'amy';
```



参考：[前缀索引，一种优化索引大小的解决方案](https://www.cnblogs.com/studyzy/p/4310653.html)

## 前缀索引和索引选择性

前缀索引：说白了就是对文本的前几个字符来建立索引，这样建立起来的索引更小，查询更快。

缺点：不能在 ORDER BY 或 GROUP BY 中使用前缀索引，也不能把它们用作覆盖索引



索引选择性：数据不重复的个数与总个数的比值



如何选择前缀的长度？

1. 计算字段的索引选择性：

    ```sql
    select 1.0*count(distinct column_name)/count(*)
    from table_name
    ```

2. 计算前缀的索引选择性：

   ```sql
   select 1.0*count(distinct left(column_name,前缀长度))/count(*)
   from Employee
   ```


3. 尽量让前缀的索引选择性贴近整个字段的索引选择性

参考：[前缀索引，一种优化索引大小的解决方案](https://www.cnblogs.com/studyzy/p/4310653.html)

# 锁与事务篇

## 事务的四大特性 ACID

- 原子性(Atomicity)：组成事务的操作要么全部成功，要么全部失败回滚，不能部分成功部分失败；

- 一致性(Consistency)：事务在执行前和执行后都要使数据保持一致性状态。

  拿转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。

- 隔离性(Isolation)：事务之间的操作相互隔离，互不干扰。

- 持久性(Durability)：事务一旦提交，事务的操作必须持久化到硬盘上，即便数据库遇到故障也不会丢失已提交事务的操作

参考：[数据库事务的四大特性以及事务的隔离级别](https://www.cnblogs.com/fjdingsd/p/5273008.html)

## 脏读、不可重复读、幻读

- 脏读：指一个事务读到其它事务未提交的修改。

- 不可重复读：指一个事务对某项记录进行多次查询得到了不同的值，这是因为在查询过程中被其它事务修改并提交了。

  不可重复读和脏读的区别是，“脏读”是读到了其它事务未提交的脏数据，而“不可重复读”则是读到了其它事务已提交的数据。

- 幻读|虚读：指一个事务进行多次查询得到了更多的数据，这是因为在查询过程中其它事务插入了新数据。这就会导致当前事务对这些数据处理后，再次查询会发现还有一些数据没处理，就像发生了幻觉一样，这就是幻读。

  ”幻读“和”不可重复读“都是读取了另一个已提交事务的数据而出现的问题（这点就脏读不同），所不同的是”不可重复读“是对某项数据多次查询得到了不同的值，而”幻读“是对某批数据多次查询得到了更多的数据。



参考：

https://www.cnblogs.com/fjdingsd/p/5273008.html

https://blog.csdn.net/qq_33290787/article/details/51924963

## 事务的隔离级别

- 读未提交（Read Uncommitted）：事务可以读到其它事务未提交的**修改**（注意这里的修改指的是更新操作，而不是插入删除操作，下同）。所以会出现**脏读**。
- 读已提交（Read Committed）：事务只能读到其他事务已提交的修改。**解决了脏读的问题，但是因为会读到其它事务已提交的“修改”（不是新增），所以会出现不可重复读的问题。**
- 可重复读（Repeatable Read）：这是 MySQL 默认隔离级别。事务无法读到其它事务已提交或未提交的**“修改”（不是新增）**，保证了事务在执行过程中读取到的数据前后一致。**解决了脏读、不可重复读的问题，但是会出现幻读的问题。** 
- 可串行化|可序列化（Serializable）：最高隔离级别。所有事务**按顺序串行化执行**，会对整个表加锁（不管是读还是写），后一个的事务必须等前一个事务执行完成，才能继续执行。**解决了脏读、不可重复读、幻读的问题，但是性能较低**

## *InnoDB 是如何实现 ACID ？

## *MVCC 机制

MVCC (Multi Version Concurrency Control) 即**多版本并发控制**，MySQL 的默认隔离级别“可重复读”就是通过 MVCC 去实现的。

**MVCC 实现原理**

MVCC是通过在每行记录后面保存两个隐藏的列来实现的。这两个列，一个保存了行的创建时间，一个保存行的过期时间（或删除时间）。这个时间并不是实际的时间值，而是系统版本号（system version number)。每开始一个新的事务，系统版本号都会自动递增，作为当前事务的版本号，用来和查询到的每行记录的版本号进行比较。

MySQL 的 InnoDB 引擎在 Repeatable Read 隔离级别下不会产生幻读，得益于 MVCC 机制

作者：浆水面韭菜花
链接：https://www.jianshu.com/p/f692d4f8a53e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 表级锁，行级锁，页级锁

- 表级锁：锁开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低；
- 行级锁：锁开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高；  
- 页面锁：锁开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

参考：[MySQL锁详解](https://www.cnblogs.com/luyucheng/p/6297752.html)

## MySQL 死锁

死锁是指两个或两个以上的进程因争夺资源而造成的一种互相等待的现象。

造成死锁的关键在于两个或多个 MySQL 会话加锁的顺序不一样。



查看死锁的命令：

```sql
show engine innodb status\G
```



参考：https://blog.csdn.net/tr1912/article/details/81668423

## 什么时候行锁会变为表锁？

索引失效的时候。

InnoDB 的行锁是加在索引上的，如果索引失效的话，就会升级为表锁。

- 无锁

  ```sql
  # 主键不存在
  select * from user where id = -1 for update;
  ```

- 行锁

  ```sql
  select * from user where id = 1 for update;
  select * from user where id = 1 and name = 'lucy' for update;
  ```

- 表锁

  ```sql
  # 主键不明确
  select * from user where name = 'lucy' for update;
  ```

## Bin Log，Redo Log，Undo Log

- Bin Log：MySQL 服务层产生的日志，常用来进行数据恢复、数据库复制、同步。MySQL 主从架构，就是采用 slave 同步 master 的binlog 实现的

- Redo Log：记录了数据操作在物理层面的修改，mysql中使用了大量缓存，修改操作时会直接修改内存，而不是立刻修改磁盘，事务进行中时会不断的产生redo log，在事务提交时进行一次flush操作，保存到磁盘中。当数据库或主机失效重启时，会根据redo log进行数据的恢复，如果redo log中有事务提交，则进行事务提交修改数据。
- Undo Log：除了记录redo log外，当进行数据修改时还会记录undo log，undo log用于数据的撤回操作，它记录了修改的反向操作，比如，插入对应删除，修改对应修改为原来的数据，通过undo log可以实现事务回滚，并且可以根据undo log回溯到某个特定的版本的数据，实现MVCC



作者：浆水面韭菜花
链接：https://www.jianshu.com/p/f692d4f8a53e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


# 优化篇



## 开启慢查询日志

参考：[MySQL慢查询（一） - 开启慢查询](https://www.cnblogs.com/luyucheng/p/6265594.html)



## *Using temporary; Using filesort 优化



## 常见的 SQL 优化

SQL 优化的中心思想是查询的时候尽量使用索引，避免全表扫描。SQL 性能优化的目标：至少要达到 range 级别，要求是 ref 级别，如果可以是 consts 最好。 type 不能是 index 

1. 优化第一步，给频繁查询的字段添加索引。在 where 和 order by 涉及到的字段上添加索引。

2. *避免在 where 子句中使用 is null 或 is not null 对字段进行判空。

   如：

   ```sql
   select id from table where name is null
   ```

   在这个查询中，就算我们为 name 字段设置了索引，查询分析器也不会使用，因此查询效率底下。为了避免这样的查询，在数据库设计的时候，应避免在建立索引的列中使用空值。这里如果我们将 name 字段的默认值设置为0，那么我们就可以这样查询

   ```sql
   select id from table where name = 0
   ```
   
3. *避免在 where 子句中使用 != 或 <> 操作符。

   如：

   ```
   select name from table where id <> 0
   ```

   数据库在查询时，对 != 或 <> 操作符不会使用索引，而对于 < 、 <= 、 = 、 > 、 >= 、 BETWEEN AND，数据库才会使用索引。因此对于上面的查询，正确写法应该是：

   ```
   select name from table where id < 0
   union all
   select name from table where id > 0
   ```

   这里我们为什么没有使用 or 来链接 where 后的两个条件呢？这就是我们下面要说的优化技巧。  

4. *避免在 where 子句中使用 or 来链接条件，应该使用 union all。

   如：

   ```sql
   select id from table where name = 'UncleToo' or name = 'PHP'
   ```

   这种情况，我们可以这样写：

   ```
   select id from table where name = 'UncleToo'
   union all
   select id from table where name = 'PHP'  
   ```

5. 少用 in 或 not in，多用 between，实在不行也要把 in 后边的集合元素数量控制在 1000 个之内。

   虽然对于 in 的条件会使用索引，但是在某些特定的情况，使用其他方法也许效果更好。如：

   ```
   select name from table where id in (1,2,3,4,5)
   ```

   像这种连续的数值，我们可以使用 BETWEEN AND，如：

   ```
   select name from table where id between 1 and 5  
   ```

6. 避免在 where 子句中对字段进行表达式操作。

   如：

   ```
   select name from table where id/2 = 100
   ```

   正确的写法应该是：

   ```
   select name from table where id = 100*2  
   ```

   

   如果是这种查询会用到索引吗？

   ```sql
   select * from table where (a_num + b_num) > 0
   ```

   

7. 避免在 where 子句中对字段进行函数操作。

   如：

   ```
   select id from table where substring(name,1,8) = 'UncleToo'
   ```

   或

   ```
   select id from table where datediff(day,datefield,'2014-07-17') >= 0
   ```

   这两条语句中都对字段进行了函数处理，这样就是的查询分析器放弃了索引的使用。正确的写法是这样的：

   ```
   select id from table where name like 'UncleToo%'
   ```

   或

   ```
   select id from table where datefield <= '2014-07-17'
   ```

   也就是说，不要在 where 子句中的 = 左边进行函数、算术运算或其他表达式运算。  

8. 在子查询中，用 exists 代替 in 是一个好的选择。

   如：

   ```
   select name from a where id in(select id from b)
   ```

   如果我们将这条语句换成下面的写法：

   ```
   select name from a where exists(select 1 from b where id = a.id)
   ```

   这样，查询出来的结果一样，但是下面这条语句查询的速度要快的多。  

   另外需要注意的是，IN 适合内表小而外表大的情况，EXISTS 适合内表大而外表小的情况。

   **PS：想一想为什么 exists 子句使用 select 1 代替 select id，这样做有什么好处？**

9. 使用 not exists 代替 not in

   如果查询语句使用了not in 那么内外表都进行全表扫描，没有用到索引；

   而 not exists 的子查询依然能用到表上的索引。

   所以无论那个表大，用 not exists 都比 not in 要快。

10. 模糊查询时禁止使用左模糊和全模糊。

   下面的语句会导致全表扫描，尽量少用。如：

   ```
   select id from table where name like '%UncleToo%'
   ```

   或者

   ```
   select id from table where name like '%UncleToo'
   ```

   而下面的语句执行效率要快的多，因为它使用了索引：

   ```
   select id from table where name like 'UncleToo%'
   ```

11. 当只要一行数据时使用 LIMIT 1。

    数据库会在查找到一条数据后停止搜索，不会继续往后查询下一条符合条件的数据。

12. 使用 varchar 代替 char 。因为变长字段存储空间小，可以节省存储空间。

参考：

https://blog.csdn.net/jie_liang/article/details/77340905

https://blog.csdn.net/csdnstudent/article/details/40398245

## IN 和 EXISTS 的区别

见上面的SQL优化

## NOT IN 和 NOT EXISTS 的区别

见上面的SQL优化



## 分页查询优化

分页查询的语句一般如下：

```sql
SELECT * FROM table LIMIT [offset,] rows 
SELECT * FROM table rows OFFSET offset
-- offset 和 rows 都是参数
```

如查询100000条数据后面的100条：

```sql
SELECT * FROM table LIMIT 100000, 100
SELECT * FROM table 100 OFFSET 100000
```

分页查询的本质其实是先查询 offset + rows 条数据，然后再剔除前面的 offset 条数据，所以分页查询的偏移量如果过大，查询效率会显著下降。**如果表的主键ID是自增的话，则可以做一定的优化**。

```
select * from table where id >= (select id from table limit 100000,1) limit 100;
```

## count(*) 的实现方式

不同的存储引擎，count(*) 有不同的实现方式。

- MyISAM 引擎：将表的总行数存到磁盘上，执行 count(*) 的时候会直接返回（没有 where 子句的情况下）；

- InnoDB 引擎：需要把对当前会话可见的行读出来作统计。

  为什么会这样子呢？这和 InnoDB 使用 MVCC 实现事务的可重复读隔离级别有关。每一行记录都要判断自己是否对当前会话可见，因此对于 count(*) 请求来说，InnoDB 只好把数据一行一行地读出依次判断，可见的行才能够用于计算“基于这个查询”的表的总行数。

## 不同 count 区别和用法

count(*)、count(主键 id)、count(字段) 和 count(1) 的区别和用法？

先说结论，按照效率排序的话，count(字段)<count(主键 id)<count(1)≈count(\*)，所以尽量使用 count(*)。



参考：[极客时间《MySQL实战45讲》14 | count(*)这么慢，我该怎么办？](https://time.geekbang.org/column/article/72775)

## 如何存储IP地址？

对于 IPv4，它是32位的，只需要4个字节即可存储，正好对应 MySQL 的 unsigned int(10) 数据类型

```sql
SELECT INET_ATON('192.168.1.1'); #结果为 3232235777
SELECT INET_NTOA('3232235777'); #结果为 192.168.1.1
```

对于 IPv6，它是128位的，每16位用分号分割，总共需要16字节来存储，建议使用 char(32) 定长数据类型来存储

```sql
SELECT HEX(INET6_ATON('ABCD:EF01:2345:6789:ABCD:EF01:2345:6789')); #去掉分号后长度为32位
SELECT INET6_NTOA(UNHEX('ABCDEF0123456789ABCDEF0123456789')); 
```

参考：[MySQL如何有效的存储IP地址？](https://mp.weixin.qq.com/s/P_wN_UwYnEPtbCwHTHh_Bg)

# SQL 语句篇

## SQL 删除重复数据只保留一条

SQL 通常情况下会存在重复数据，如果这些重复数据除了主键字段不一样，其它字段的数据都一样，可以使用以下方法删除重复数据并只保留一条。如果所有字段的数据都一样，包括主键字段，那就无可奈何了。

主要方法是：先根据重复字段给数据分组，此时重复数据都会在同一组中，然后再找到分组数据中的最大或最小的主键ID，最后再删除主键ID不在该范围的数据，即可把重复的数据删除。

```sql
DELETE FROM 表名 WHERE 主键ID NOT IN ( SELECT * FROM ( SELECT MAX(主键ID) FROM 表名 GROUP BY 重复的字段 ) AS b);
```

上面的 MAX 可以换成 MIN。

实例：

demo 表有三个字段 id、uname、age，其中 id 是主键索引，数据库中存在有 uname 和 age 都相同的重复数据，需要删除重复数据且只保留重复数据中的一条。

```sql
CREATE TABLE IF NOT EXISTS `demo` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uname` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `age` tinyint(4) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

-- 正在导出表  db_example.demo 的数据：~7 rows (大约)
DELETE FROM `demo`;
/*!40000 ALTER TABLE `demo` DISABLE KEYS */;
INSERT INTO `demo` (`uname`, `age`) VALUES
	('john', 11),
	('john', 11),
	('john', 11),
	('john', 13),
	('amy', 12),
	('amy', 12),
	('brance', 14),
	('candy', 10);
```

执行以下语句可以删除并保留一条重复数据：

```sql
DELETE FROM demo WHERE id NOT IN ( SELECT * FROM ( SELECT MAX(id) FROM demo GROUP BY uname, age ) AS b );
```

## 生成随机字符串或随机数字

1. 生成20位随机字符串示例：

   ```sql
   SELECT SUBSTRING(MD5(RAND()), 1, 20)
   ```

   - RAND() 函数生成随机大于0小于1的随机数字
   - MD5() 函数生成指定参数的MD5值
   - SUBSTRING() 函数截取字符串

2. 生成3位随机数字示例：

   ```sql
   SELECT CEILING(RAND() * 10) # 生成1至N之间的随机数
   SELECT FLOOR(RAND() * 10) # 生成0至N-1之间的随机数
```
   
   - FLOOR() 函数向下取整
- CEIL() 函数向上取整，FLOOR() 可以替换为 CEIL()
   
3. 生成指定范围的数字：

   ```sql
   SELECT ROUND(((MAX_VALUE - MIN_VALUE) * RAND() + MIN_VALUE)) 
   # 生成大于等于MIN_VALUE，小于等于MAX_VALUE的数字
   ```

   

## 查询学生总分和各科成绩

表结构：

```sql
CREATE TABLE `result` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(50) NOT NULL,
	`course` VARCHAR(50) NOT NULL,
	`score` INT(11) NOT NULL DEFAULT '0',
	PRIMARY KEY (`id`)
)ENGINE=InnoDB;
```

表数据：

```sql
DELETE FROM `result`;
INSERT INTO `result` (`id`, `name`, `course`, `score`) VALUES
	(1, '张三', '语文', 78),
	(2, '张三', '数学', 83),
	(3, '李四', '语文', 88),
	(4, '李四', '英语', 89),
	(5, '王二', '语文', 80),
	(6, '王二', '数学', 76),
	(7, '王二', '英语', 84);
```

要求查询结果如下：

| 姓名 | 语文 | 数学 | 英语 | 总分 |
| ---- | ---- | ---- | ---- | ---- |
| 张三 | 78   | 83   | 0    | 161  |
| 李四 | 88   | 0    | 89   | 177  |
| 王二 | 80   | 76   | 84   | 240  |

查询语句：

```sql
SELECT r.name AS '姓名', IFNULL(SUM(r.score), 0) AS '总分', IFNULL(t1.score, 0) AS '语文', IFNULL(t2.score, 0) AS '数学', IFNULL(t3.score, 0) AS '英语' FROM result r 
LEFT JOIN (
	SELECT NAME, score FROM result WHERE course = '语文'
) t1 ON t1.name = r.name 
LEFT JOIN (
	SELECT NAME, score FROM result WHERE course = '数学'
) t2 ON t2.name = r.name 
LEFT JOIN (
	SELECT NAME, score FROM result WHERE course = '英语'
) t3 ON t3.name = r.name 
GROUP BY r.name 
```

# MySQL 8.0 新特性

参考：[慕课网《玩转MySQL8.0新特性》](https://www.imooc.com/learn/1102)

## 角色

MySQL 8.0 引入了角色功能，其本质上是通过用户来模拟角色的效果。

```sql
create role 'select_role'; # 创建角色
select user,host,authentication_string from mysql.user; # 查询用户，这里会发现创建角色其实就是创建用户
grant select on db.tbl to 'select_role'; # 授予角色权限
show grants for 'select_role'; # 查询角色权限
grant 'select_role' to 'u_user'; # 授予用户角色
show grants for 'u_user' using 'select_role'; # 查询用户权限
set default role 'select_role' to 'u_user'; # 设置默认使用权限
set default role all to 'u_user'; # 设置默认使用所有权限

# 查询默认使用角色
select * from mysql.default_roles;
select * from mysql.role_edges;

# 重新登录后
select current_role(); # 查询当前用户当前角色
set role 'select_role'; # 设置使用的权限

# 撤销角色权限
revoke select on db.tbl from 'select_role';
```



## 索引篇

- 隐藏索引：将索引隐藏起来，查询的时候就不会使用该索引，但是索引还是会维护。隐藏索引的作用是用来表示索引软删除，避免在调试时反复删除索引和重新建立索引，因为在表数据比较大的情况下重新建立索引需要消耗资源。

  ```sql
  create index idx_col on tbl(col) invisible;
  alter table tbl alter index idx_col invisible;
  ```

  

- 降序索引：索引通常是升序的，也可以建立降序的

  ```sql
  create index idx_col on tbl(col desc);
  ```

  

- 函数索引：索引可以是函数或者表达式，本质上就是虚拟列+索引

  ```sql
  create index idx_total on tbl( (col1 + col2) );
  ```

  此法等同于

  ```sql
  alter table tbl add column total generated as (col1 + col2);
  create index idx_total on tbl( total );
  ```

  

## 通用表表达式（CTE）

通用表表达式，英文名 Common Table Expression，缩写 CTE，又叫 WITH 子句。

### 非递归 CTE

派生表：

```sql
select * from (select 1) as dt;
```

通用表表达式：

```sql
with cte as (select 1)
select * from cte;
# cte 是派生表表名，随便起
```

上面两种写法结果都是一样的。

### 递归 CTE

最简单的递归 CTE 示例，生成1-5：

```sql
with recursive cte(n) as
(
    select 1
    union all
    select n+1 from cte where n<5
)
select * from cte;
# 这里的 cte 是递归派生表的表名，n 为派生表查询字段。
# union all 前面的 select 1 为递归的初始化语句
# 后面的为递归语句
```

上述语句可理解为如下伪语句：

```sql
select * from
(
    with cte(n) as
    (
        select 1
    )
    select n+1 from cte where n<5
    # 相当于循环执行非递归CTE 5遍
)

```



执行结果如下：

```sql
+------+
| n    |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
|    5 |
+------+
5 rows in set (0.00 sec)

```



这里举个员工表的查询为例。manager_id 表示上级 id，为空表示当前员工是最高级

```sql
create table employees (id int unsigned not null auto_increment primary key, name varchar(50) not null, manager_id int unsigned);

insert into employees(id, name, manager_id) values
(29, 'Pedro', 198),
(72, 'Pierre', 29),
(123, 'Adil', 692),
(198, 'John', 333),
(333, 'Yasmina', NULL),
(692, 'Tarek', 333),
(4610, 'Sarah', 29);
```

使用递归来查询上下级关系：

```sql
WITH recursive employees_path(id, name, path) AS 
(
	SELECT id, name, CAST(id AS CHAR(200))
	FROM employees
	WHERE manager_id IS NULL
	UNION ALL
	SELECT e.id,e.name, CONCAT(ep.path, ',', e.id) AS path
	FROM employees_path ep
	JOIN employees e ON e.manager_id = ep.id
)
SELECT *
FROM employees_path
ORDER BY path;
# union all 前面的为递归的初始化语句，后面的为递归语句
# CAST(id AS CHAR(200)) 语句是将 id 字段映射为 path 字段
```



### 练习 - SQL 生成斐波那契数列

尝试使用递归CTE生成斐波那契数列（一个数等于前两个数之和）：`0,1,1,2,3,5,8...`

```sql
WITH recursive cte (id, curr, pre) AS 
(
    SELECT 
      1 AS id,
      0 AS curr,
      0 AS pre 
    UNION ALL 
    SELECT 
      id + 1,
      IF(id < 2, 1, curr + pre),
      curr 
    FROM
      cte 
    WHERE id < 10
)
SELECT id AS n,curr AS f 
FROM cte ;
```

递归表结果如下：

```sql
+------+------+------+
| id   | curr | pre  |
+------+------+------+
|    1 |    0 |    0 |
|    2 |    1 |    0 |
|    3 |    1 |    1 |
|    4 |    2 |    1 |
|    5 |    3 |    2 |
|    6 |    5 |    3 |
|    7 |    8 |    5 |
|    8 |   13 |    8 |
|    9 |   21 |   13 |
|   10 |   34 |   21 |
+------+------+------+
10 rows in set (0.00 sec)
```



参考

[SQL 生成斐波那契数列](https://zhuanlan.zhihu.com/p/140081748)

## 窗口函数

窗口函数多用于数据统计、报表，跟分组聚合类似