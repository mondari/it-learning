[TOC]

# 结果映射（resultMap）

resultMap 的作用是将数据表的列值映射到对象的属性上，从而达到将SQL查询的结果集 ResultSet 映射到 Java对象的目的。

resultType 的本质就是 resultMap。



resultMap（结果映射）的元素如下：

- id 和 result：结果映射的最基本元素，都是将列值映射到对象的属性上。不同的是 id 元素表示对象的标识属性，这样可以提高整体的性能。
- constructor（构造方法）：结果映射时，如果Java对象没有无参数构造方法，则的通过该元素使用有参数构造方法
  - `idArg` -对象的构造方法ID入参
  - `arg` - 对象的构造方法非ID入参
- association（关联）：级联对象一对一关联
  - association的嵌套select查询
  - association的嵌套结果映射
- collection（集合）：级联对象一对多关联（多对多关联需要拆分成两个一对多关联）
  - collection的嵌套select查询
  - collection的嵌套结果映射
- discriminator（鉴别器）：根据结果值来决定使用哪个 `resultMap`



注意：

`javaType="int"` 表示 Integer 类型，`javaType="_int"` 表示 int 类型



# 懒加载解决N+1问题

什么是“N+1查询问题”（N+1 Selects Problem）：

- 执行一条SQL查询语句来查询一列记录（+1）
- 对返回的每条记录，又分别执行一条SQL查询语句去加载其详细信息（N）

这个问题会导致成百上千的 SQL 查询语句被执行。有时候，我们不希望产生这样的后果，只希望在获取记录的详细信息时才去执行SQL查询语句，这就需要“懒加载”去解决。



MyBatis开启懒加载：

1. lazyLoadingEnabled 设为 true。懒加载全局配置参数
2. aggressiveLazyLoading 设为 false。false表示按需加载。true表示懒加载的对象可能被任何懒属性全部加载
3. association 和 collection 中的 fetchType 设为 lazy。设置该属性会在映射中忽略全局配置参数 `lazyLoadingEnabled`



# 缓存

- 一级缓存是**SqlSession级别**的缓存，默认开启。
- 二级缓存是**Mapper级别**的缓存，定义在Mapper文件的\<cache>标签中，并需要开启全局二级缓存cacheEnabled，多个Mapper文件可以共用缓存，依赖\<cache-ref>标签配置

注意：使用二级缓存的前提，**必须保证所有的增删改查都在同一个命名空间下才行。**

或者这样理解，二级缓存是保存在Mapper对象中的，现在有一张user表，定义了两个Mapper文件（一般是一个表定义一个Mapper文件，这里举个特殊情况），AMapper.xml和BMapper.xml，B修改了user表中的内容，A是感知不到的，那么再从A里查询如果用到了缓存，就是旧的数据。

# 枚举与TypeHandler



# foreach

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

# 批量操作

## 批量插入

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



## 批量更新

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

