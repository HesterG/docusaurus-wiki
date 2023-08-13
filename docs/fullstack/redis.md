---
title: Redis
---

## 什么是Redis

Redis 是完全开源的，遵守 BSD 协议，是一个高性能的 key-value 数据库。

Redis 与其他 key - value 缓存产品有以下三个特点：

- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份

Redis 优势

- 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
- 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
- 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

## Redis 为什么这么快

[zhihu](https://zhuanlan.zhihu.com/p/460236324)

## Redis vs MySQL

使用场景区别 

1. mysql支持sql查询，可以实现一些关联的查询以及统计； 

2. redis对内存要求比较高，在有限的条件下不能把所有数据都放在redis； 

3. mysql偏向于存数据，redis偏向于快速取数据，但redis查询复杂的表关系时不如mysql，所以可以把热门的数据放redis，mysql存基本数据。
