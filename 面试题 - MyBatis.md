[TOC]

## 缓存机制

MyBatis 支持两级缓存，分别如下：

- 一级缓存是 **SqlSession** 级别的缓存，默认是开启的。

  **在同一个数据库会话中（也就是一个 SqlSession 中）**，如果执行多条完全相同的查询语句，会优先命中一级缓存，避免直接对数据库进行查询，提高性能。

  **注意事项**：在同一个数据库会话中，对数据库进行修改操作，会导致一级缓存失效。

  **缺点**：在多个数据库会话场景中，如果一个会话对数据库进行修改操作，另一个会话会有脏数据的问题，因为它感知不到（解决方法是将一级缓存的级别由 session 改为 statement）。

  

- 二级缓存是 **命名空间** 级别的缓存，能够实现多个 `SqlSession` 之间共享缓存。需要手动开启。我们通常是一个单表对应一个命名空间

  **注意事项**：在同一个命名空间中，对数据库进行修改操作，会导致二级缓存失效。

  **缺点**：多表查询时会出现脏数据（解决方法：添加 `<cache-ref>` 标签配置多个映射文件公用同一块缓存）。

  **开启方式**：开启全局二级缓存 cacheEnabled，然后在映射文件中添加 `<cache>` 标签。多个命名空间可以共用缓存，依赖 `<cache-ref>` 标签的配置

  

建议在生产环境中关闭 MyBatis 的缓存功能，单纯作为一个ORM框架使用。

参考：[聊聊MyBatis缓存机制](https://tech.meituan.com/2018/01/19/mybatis-cache.html)

## #{}和${}的区别是什么(待优化)

- `${}` 占位符：只是简单地将该占位符进行静态文本替换，会有 SQL 注入的风险
- `#{}` 占位符：MyBatis 会将其解析为 `?` ，在 SQL 执行前会使用 PreparedStatement 进行参数设置（通过反射的机制获取参数对象的属性值），可以防止 SQL 注入。

参考：https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/mybatis/mybatis-interview

## 通常一个 XML 映射文件，都会写一个 Dao 接口与之对应，其工作原理是什么？Dao 接口里的方法参数不同时，方法能重载吗？

Dao 接口，就是人们常说的 `Mapper` 接口，接口的全限名，就是映射文件中的 `namespace` 的值，接口的方法名，就是映射文件中`MappedStatement` 的 id 值，接口方法内的参数，就是传递给 sql 的参数。在 XML 映射文件中，每一个 `<select>`、`<insert>`、`<update>`、`<delete>` 标签，都会被解析为一个 `MappedStatement` 对象。

`Mapper` 接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一 个`MappedStatement` 。举例：`com.mybatis3.mappers.StudentDao.findStudentById`，可以唯一找到 namespace 为 `com.mybatis3.mappers.StudentDao` 下面 `id = findStudentById` 的 `MappedStatement` 。

**Dao 接口的工作原理**是 JDK 动态代理，Mybatis 运行时会使用 JDK 动态代理为 Dao 接口生成代理对象，代理对象会拦截接口方法，转而执行 `MappedStatement` 所代表的 sql 语句，然后将 sql 执行结果返回。

**Dao 接口里的方法，是不能重载的**，因为是通过全限名+方法名的保存和寻找相应的 `MappedStatement` 。



# 非面试题，纯笔记

## 结果映射（resultMap）

resultMap 的作用是将数据表的列值映射到对象的属性上，从而达到将SQL查询的结果集 ResultSet 映射到 Java对象的目的。

resultType 的本质就是 resultMap。



resultMap（结果映射）的元素如下：

- id 和 result：结果映射的最基本元素，都是将列值映射到对象的属性上。不同的是 id 元素表示对象的标识属性，这样可以提高整体的性能。
- constructor（构造方法）：结果映射时，如果Java对象没有无参数构造方法，则的通过该元素使用有参数构造方法
  - `idArg` -对象的构造方法ID入参
  - `arg` - 对象的构造方法非ID入参
- association（关联）：级联对象一对一关联
  - association的嵌套select查询
  - association的嵌套结果映射（不推荐）
- collection（集合）：级联对象一对多关联（多对多关联需要拆分成两个一对多关联）
  - collection的嵌套select查询
  - collection的嵌套结果映射（不推荐）
- discriminator（鉴别器）：根据结果值来决定使用哪个 `resultMap`



注意：

`javaType="int"` 表示 Integer 类型，`javaType="_int"` 表示 int 类型



## 懒加载解决N+1问题

什么是“N+1查询问题”（N+1 Selects Problem）：

- 执行一条SQL查询语句来查询一列记录（+1）
- 对返回的每条记录，又分别执行一条SQL查询语句去加载其详细信息（N）

这个问题会导致成百上千的 SQL 查询语句被执行。有时候，我们不希望产生这样的后果，只希望在获取记录的详细信息时才去执行SQL查询语句，这就需要“懒加载”去解决。



MyBatis开启懒加载：

1. lazyLoadingEnabled 设为 true。懒加载全局配置参数
2. aggressiveLazyLoading 设为 false。false表示按需加载。true表示懒加载的对象可能被任何懒属性全部加载
3. association 和 collection 中的 fetchType 设为 lazy。设置该属性会在映射中忽略全局配置参数 `lazyLoadingEnabled`



## 枚举与TypeHandler



## foreach

Mapper代码示例：

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

相当于SQL语句：

```sql
select * from post p
where id in 
(item1, item2, item3, ...)
```

**注意** foreach 中的 collection 属性值可以是 list、set和map。如果是list和set，则 index 是当前迭代的次数，item 的值是本次迭代获取的元素；如果是map，则 index 是键，item 是值

## 批量操作

### 批量插入

Mapper代码示例：

```xml
<insert id="batchInsert" parameterType="java.util.List">
    INSERT INTO 
    table(field1, field2, field3)
    VALUES
    <foreach collection="list" item="item" index="index" separator=",">
        (#{item.value1}, #{item.value2}, #{item.value3})
    </foreach>
</insert>
```

相当于SQL语句：

```sql
INSERT INTO table (field1,field2,field3) VALUES ('valueA1',"valueB1","valueC1"),('valueA2',"valueB2","valueC2"),('valueA3',"valueB3","valueC3");
```



### 批量更新

Mapper代码示例：

```xml
<update id="updateXXX" parameterType="java.util.List">
    UPDATE table SET 
    field1 = 
    <foreach collection="list" item="item"  index="index"  separator=" " open="case `name`"  close="end,">
        when #{item.name} then #{item.data}
    </foreach>
    field2 = 
    <foreach collection="list" item="item"  index="index"  separator=" " open="case `name`"  close="end">
        when #{item.name} then #{item.record}
    </foreach>
    WHERE
    name in
    <foreach collection="list" item="item" index="index" open="(" separator="," close=" )">
        #{item.name}
    </foreach>
</update>
```

相当于SQL语句：

```sql
UPDATE table 
    SET 
    field1 = CASE id 
        WHEN 1 THEN 'valueA1'
        WHEN 2 THEN 'valueA2'
        WHEN 3 THEN 'valueA3'
    END, 
    field2 = CASE id 
        WHEN 1 THEN 'valueB1'
        WHEN 2 THEN 'valueB2'
        WHEN 3 THEN 'valueB3'
    END
WHERE id IN (1,2,3)
```

