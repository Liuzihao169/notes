###  一、redis面试题

- 什么是 Redis？
- Redis 有哪些优缺点？
- 为什么要用 Redis / 为什么要用缓存？
- 为什么要用 Redis 而不用 map/guava 做缓存?
- Redis 为什么这么快？

## **数据类型**

- Redis 有哪些数据类型？
- Redis 的应用场景？

## **持久化**

- 什么是 Redis 持久化？
- Redis 的持久化机制是什么？各自的优缺点？
- 如何选择合适的持久化方式
- Redis 持久化数据和缓存怎么做扩容？
- 过期键的删除策略
- Redis 的过期键的删除策略
- Redis key 的过期时间和永久有效分别怎么设置？
- 我们知道通过 expire 来设置 key 的过期时间，那么对过期的数据怎么处理呢?

## **内存相关**

- MySQL 里有 2000w 数据，redis 中只存 20w 的数据，如何保证 redis 中的数据都是热点数据
- Redis 的内存淘汰策略有哪些
- Redis 主要消耗什么物理资源？
- Redis 的内存用完了会发生什么？
- Redis 如何做内存优化？

## **线程模型**

- Redis 线程模型

## **事务**

- 什么是事务？
- Redis 事务的概念
- Redis 事务的三个阶段
- Redis 事务相关命令
- 事务管理（ACID）概述
- Redis 事务支持隔离性吗
- Redis 事务保证原子性吗，支持回滚吗
- Redis 事务其他实现

## **集群方案**

- 哨兵模式
- 官方 Redis Cluster 方案(服务端路由查询)
- 基于客户端分配
- 基于代理服务器分片

## **Redis 主从架构**

- Redis 集群的主从复制模型是怎样的？
- 生产环境中的 redis 是怎么部署的？
- 说说 Redis 哈希槽的概念？
- Redis 集群会有写操作丢失吗？为什么？
- Redis 集群之间是如何复制的？
- Redis 集群最大节点个数是多少？
- Redis 集群如何选择数据库？

## **分区**

- Redis 是单线程的，如何提高多核CPU的利用率？
- 为什么要做 Redis 分区？
- 你知道有哪些 Redis 分区实现方案？
- Redis 分区有什么缺点？

## **分布式问题**

- Redis 实现分布式锁
- 如何解决 Redis 的并发竞争 Key 问题
- 分布式 Redis 是前期做还是后期规模上来了再做好？为什么？
- 什么是 RedLock

## **缓存异常**

- 缓存雪崩
- 缓存穿透
- 缓存击穿
- 缓存预热
- 缓存降级
- 热点数据和冷数据
- 缓存热点 key

## **常用工具**

- Redis 支持的 Java 客户端都有哪些？官方推荐用哪个？
- Redis 和 Redisson 有什么关系？
- Jedis 与 Redisson 对比有什么优缺点？

## **其他问题**

- Redis 与 Memcached 的区别
- 如何保证缓存与数据库双写时的数据一致性？
- Redis 常见性能问题和解决方案？
- Redis 官方为什么不提供 Windows 版本？
- 一个字符串类型的值能存储最大容量是多少？
- Redis 如何做大量数据插入？
- 假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某个固定的已知的前缀开头的，如果将它们全部找出来？
- 使用 Redis 做过异步队列吗，是如何实现的
- Redis 如何实现延时队列
- Redis 回收进程如何工作的？
- Redis 回收使用的是什么算法？
- redis面试题

https://note.youdao.com/ynoteshare1/index.html?id=686fed984828187d452f40a55f5e6eed&type=note