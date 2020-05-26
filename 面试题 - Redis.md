[TOC]

## Redis 特点

单线程+IO多路复用

## Redis 的数据结构及其使用场景

- string 字符串：普通的键值对数据
- list 双向链表：消息队列、评论列表、时间轴数据
- set 集合：Redis 非常人性化的为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同好友、共同关注、共同爱好的功能
- zset 有序集合：分数排行榜（key 设置为“积分排行榜”，value 设置为“id”，score 设置为“积分”
- hash 字典：关联性强的键值对数据，用户信息（key 设置为“用户信息”，hkey 设置为”姓名“，hval 设置为”张三“）

## Redis 集群搭建

参考：https://blog.csdn.net/qq_42815754/article/details/82912130

Redis集群至少需要3个节点，因为投票容错机制要求超过半数节点认为某个节点挂了该节点才是挂了，所以2个节点无法构成集群

为了保障高可用，需要每个节点都要有个备用节点，所以之前需要6个节点才能组成一个高可用集群

Redis没有中心节点，节点之间使用PING-PONG机制进行通信

如何判断集群是否挂了呢? 如果集群中任意一个节点挂了，而且该节点没有从节点（备份节点），那么这个集群就挂了。

每个Redis集群理论上最多支持有16384个节点（即2的14次方个节点）



## 缓存穿透

大量查询不存在value的key

查询一个根本不存在的数据，缓存层和存储层都不会命中，这样就会导致每次请求都要到存储层去查询，失去了缓存保护后端存储的意义。



**解决方案**：

1. 存空值

   存储层不命中后，仍然将空对象保留到缓存层中，之后再访问这个数据将会从缓存中获取，保护了后端存储。

   **缺点**

   1. 需要更多的内存空间（解决方法：设置过期时间）
   2. 缓存层和存储层会有一段时间的数据不一致

2. 布隆过滤器

   将数据库中所有的数据的key，放入布隆过滤器中，当一个查询请求过来时，先经过布隆过滤器进行查询，如果不存在，则直接返回空；存在，则继续查；

   Redis 使用布隆过滤器伪代码：

   - 先查布隆过滤器 => 再查 Redis => 最后查数据库

   ```java
   String get(String key) {
   
       // 1.先查布隆过滤器
       if (!bloomFilter.mightContain(key)) {
           return null;
       } else {
           // 2.再查 Redis
           String value = redis.get(key);
           if (value == null) {
               // 3.最后查数据库
               value = db.get(key);
               redis.set(key, value);
           }
           return value;
       }
   
   }
   ```

   - 先查 Redis => 再查布隆过滤器 => 布隆过滤器存在则查数据库

   ```java
   String get(String key) {
       // 1.先查 Redis
       String value = redis.get(key);
       if (value == null) {
           // 2.再查布隆过滤器
           if (!bloomFilter.mightContain(key)) {
               return null;
           } else {
               // 3. 布隆过滤器存在则查数据库
               value = db.get(key);
               redis.set(key, value);
           }
       }
       return value;
   }
   ```

   

   **缺点**

   1. 需要另外维护一个集合来存放缓存的Key
   2. 布隆过滤器不支持删值操作
   3. 代码复杂度增大

   **参考**

   https://www.cnblogs.com/rinack/p/9712477.html



## 缓存击穿|缓存并发

高并发的情况下，**某个缓存**失效的瞬间，会有大量线程来重建缓存，数据库压力剧增。

解决方案：

1. 加锁：只允许一个线程重建缓存，其他线程等待重建缓存的线程执行完后，重新从缓存获取数据即可。有死锁的风险。
2. 设置缓存永不过期。逻辑上会过期，会让一个线程异步去重建缓存，缓存重建完之前输出旧值，缓存重建完之后输出新值。



## 缓存雪崩

在某一时刻发生**大量缓存**同时失效（过期或宕机），会有大量请求走向数据库，导致数据库压力剧增甚至挂掉。

注意：

- 和**缓存击穿|缓存并发**区分开，缓存雪崩是**大量缓存**失效，而缓存击穿|缓存并发是**某个缓存**失效

解决方案：

- 分散缓存的过期时间。将缓存失效时间分散开，比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。
- 使用主从备份+哨兵模式保障高可用，避免全盘崩溃
- 服务内部使用 Ehcache 本地缓存 + Hystrix 限流降级方案，保护后端数据库
- Redis 开启持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据

**参考**

https://doocs.gitee.io/advanced-java/#/./docs/high-concurrency/redis-caching-avalanche-and-caching-penetration

[阿里一面：关于【缓存穿透、缓存击穿、缓存雪崩、热点数据失效】问题的解决方案](https://mp.weixin.qq.com/s/5MloHIa5zKvYYsVVEWZjQA)

## 缓存不一致



## 布隆过滤器

**应用场景**

1. 解决缓存穿透
2. 推荐去重

**原理**

**使用布隆过滤器示例**

```java
import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels;
import java.util.ArrayList;
import java.util.List;

public class Test {

    private static int size = 1000000;

    private static BloomFilter<Integer> bloomFilter = BloomFilter.create(Funnels.integerFunnel(), size);

    public static void main(String[] args) {
        for (int i = 0; i < size; i++) {
            bloomFilter.put(i);
        }

        List<Integer> list = new ArrayList<>(1000);
        //故意取10000个不在过滤器里的值，看看有多少个会被认为在过滤器里
        for (int i = size + 10000; i < size + 20000; i++) {
            if (bloomFilter.mightContain(i)) {
                list.add(i);
            }
        }
        // 该布隆过滤器默认的误判率为0.03
        System.out.println("误判的数量：" + list.size());
        // 说明布隆过滤器中存在，并不代表真的存在，有可能实际上并不存在。
    }

}

```





## 设计一个分布式订单系统

要求优先级高的订单优先处理，并保证任意时刻一个订单只被一个线程处理，订单的数据结构该如何设计，该使用什么样的存储？

答：将要处理的订单通通存到优先级队列里，要处理的时候取出，并将其订单号存到 Redis 中作为分布式锁，保证任意时刻一个订单只能被一台机器的一个线程处理。