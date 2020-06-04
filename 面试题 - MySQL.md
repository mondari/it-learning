[TOC]



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

## MySQL 有哪些数据库引擎，区别？

MySQL 常用的两种数据库引擎：

- InnoDB：支持事务和行级锁
- MyISAM：不支持事务，只支持表级锁

## MySQL 有哪些索引？

MySQL 支持五种索引，分别是 PRIMARY，KEY，UNIQUE，FULLTEXT，SPATIAL，即主键索引，普通索引，唯一索引、全文索引和空间索引

如果从数据结构来划分的话，有四种类型：

- **B-Tree 索引**
- **Hash 索引**
- **R-Tree（空间）索引**
- **Full-text 索引**

InnoDB 和 MyISAM 支持 BTREE 和 FULLTEXT 索引，MEMORY 支持 HASH 和 BTREE 索引.

参考：[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

## 索引是怎样提高查询效率？

使用索引可以避免全表扫描

## 主键索引和非主键索引

- 主键索引的叶子节点存的是整行数据。在 InnoDB 里，主键索引也被称为聚簇索引（clustered index）。

- 非主键索引的叶子节点内容是主键的值。在 InnoDB 里，非主键索引也被称为二级索引（secondary index）。

## 事务的四大特性 ACID

- 原子性(Atomicity)：事务包含的操作要么全部成功，要么全部失败回滚

- 一致性(Consistency)：事务在执行前和执行后都要使数据保持一致性状态。

  拿转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。

- 隔离性(Isolation)：多个并发事务之间的操作相互隔离，互不干扰。

- 持久性(Durability)：事务一旦提交，事务的操作必须持久化到硬盘上，即便数据库遇到故障也不会丢失已提交事务的操作

参考：[数据库事务的四大特性以及事务的隔离级别](https://www.cnblogs.com/fjdingsd/p/5273008.html)

## 脏读、不可重复读、幻读

- 脏读：指一个事务读到其它事务未提交的记录。

- 不可重复读：指一个事务对某项记录进行多次查询得到了不同的值，这是因为在查询过程中被其它事务修改并提交了。

  不可重复读和脏读的区别是，“脏读”是读到了其它事务未提交的脏数据，而“不可重复读”则是读到了其它事务已提交的数据。

- 幻读|虚读：指一个事务对表中的**某批数据**进行处理时，另一个事务向这批数据插入（或删除？）了新数据，所以当前一个事务再次查询这批数据时，会发现多出了新数据，就像发生了幻觉一样，这就是幻读。

  ”幻读“和”不可重复读“都是读取了另一个已提交事务的数据而出现的问题（这点就脏读不同），所不同的是”不可重复读“是对某项数据多次查询得到了不同的值，而”幻读“是对某批数据多次查询得到了更多（或更少？）的数据。



参考：

https://www.cnblogs.com/fjdingsd/p/5273008.html

https://blog.csdn.net/qq_33290787/article/details/51924963

## 事务的隔离级别

- 读未提交（Read Uncommitted）：事务可以读到其它事务未提交的记录。所以会出现**脏读**。
- 读已提交（Read Committed）：事务只能读到其他事务已提交的记录。**解决了脏读的问题，但是因为会读到其它事务已提交的“修改”（不是新增），所以会出现不可重复读的问题。**
- 可重复读（Repeatable Read）：这是 MySQL 默认隔离级别。事务无法读到其它事务已提交或未提交的**“修改”（不是新增）**，保证了事务在执行过程中读取到的数据前后一致。**解决了脏读、不可重复读的问题，但是会出现幻读的问题。**
- 可串行化|可序列化（Serializable）：最高隔离级别。所有事务**按顺序串行化执行**，会对整个表加锁（不管是读还是写），后一个的事务必须等前一个事务执行完成，才能继续执行。**解决了脏读、不可重复读、幻读的问题，但是性能较低**

## SQL 优化

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

## 主从复制原理

主数据库启动二进制日志（Binary Log），记录任何数据库的改动操作，从数据库会监听主数据库的二进制日志，如果监测到变化，就会启动一个 IO 线程向主数据库请求二进制日志，主数据库会为每个从数据库的 IO 线程启动一个 Dump 线程发送二进制日志，从数据库会将二进制日志保存到本地的 Relay Log，然后启动 SQL 线程读取 Relay Log 进行同步操作。

总结：

- 主数据库会启动 Dump 线程，从数据库会启动 IO 线程和 SQL 线程
- IO 线程负责记录数据库的修改操作到二进制日志文件
- Dump 线程负责发送
- SQL 线程负责从 Relay Log 中读取日志事件并在本地重放。

主从复制后，可以实现读写分离，主库写，从库读，从库读负载均衡，降低数据库的读写压力。

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

分页查询的本质其实是先查询 offset + rows 条数据，然后再剔除前面的 offset 条数据，所以分页查询的偏移量如果过大，查询效率会显著下降。如果表的主键ID是自增的话，则可以做一定的优化。

```
select * from table where id >= (select id from table limit 100000,1) limit 100;
```

## 一条 SQL 语句是如何执行的？

连接器：管理连接，登录验证，用户名和密码是否正确
分析器：词法分析，语法分析，让MySQL知道这条SQL语句要干什么，语法有没有毛病
优化器：给出SQL语句的最优的执行方案
执行器：执行SQL语句并返回结果，但在执行前会判断是否对表有权限

## *B+树是怎样的数据结构？B树又是怎样的数据结构，两者的区别？

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
   SELECT FLOOR(RAND()*1000)
   ```

   - FLOOR() 函数向下取整
   - CEIL() 函数向上取整，FLOOR() 可以替换为 CEIL()

   

   

   

