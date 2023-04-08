# Redis

## Redis 有哪些数据类型？

- string：字符串。
- hash：哈希，键值对集合。
- list：列表，有序可重复。
- set：集合，无序去重。
- zset (sorted set)：有序集合。

## Memcached 和 Redis 有什么区别？

1. Memcached 可以使用多核；Redis 只使用单核。
2. Memcached 使用多线程的非阻塞 IO 模型；Redis 使用单线程的多路 IO 复用模型。
3. Memcached 数据结构单一，仅用来缓存数据；Redis 支持多种数据类型。
4. Memcached 不支持数据持久化，重启后数据会消失；Redis 支持数据持久化。
5. Memcached 没有提供原生的集群模式，需要依靠客户端实现往集群中分片写入数据；Redis 提供主从同步机制和集群部署能力，能够提供高可用服务。
6. Redis 速度比 Memcached 快很多。

## Redis 持久化机制有哪些？

1. RDB：根据指定规则定时将内存中的数据存储在硬盘上。是默认的持久化方案。
2. AOF：每次执行完命令后将命令记录下来。
