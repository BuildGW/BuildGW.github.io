---
title: Redis 入门
tags: [Redis]
date: 2025-06-16 13:20:30
categories:
- 数据库
- Redis
---

# Redis 入门

## Redis 是什么

Redis（REmote DIctionary Server）是一个开源的、基于内存的 Key-Value 数据库，由 Salvatore Sanfilippo 于 2009 年发布，使用 C 语言编写。与 MySQL 这类把数据落盘的关系型数据库不同，Redis 把数据放在内存里，读写延迟通常在微秒级，因此常被用作缓存、计数器、排行榜、分布式锁和轻量消息队列。

一句话概括：**Redis 是以内存为存储介质、以键值对为数据模型、支持多种数据结构的高性能数据库。**

## 为什么快

- **内存存储**：省掉磁盘 I/O，读写几乎只受网络与 CPU 影响。
- **命令执行单线程**：避免锁竞争和上下文切换（Redis 6.0 之后网络 I/O 改为多线程，但命令执行仍是单线程）。
- **高效的数据结构**：String、List、Hash、Set、ZSet 会根据数据规模自动选择更紧凑的编码。
- **I/O 多路复用**：单线程就能处理大量并发连接。

## 五种基础数据类型

| 类型 | 结构 | 典型场景 |
| --- | --- | --- |
| String | 字符串、数字、二进制 | 缓存对象、计数器、分布式锁 |
| List | 有序、可重复的双端队列 | 简单消息队列、最新列表 |
| Hash | 字段-值映射 | 存储对象的各个字段 |
| Set | 无序、去重 | 标签、共同好友、抽奖 |
| ZSet | 带分数排序的集合 | 排行榜、延时队列 |

常用命令示例：

```bash
# String：最基础的类型
SET user:1:name "饺子"
GET user:1:name
INCR article:1:views          # 原子自增，适合计数器

# List：两端插入/弹出
LPUSH queue:mail "msg1"
RPOP queue:mail

# Hash：字段-值映射，适合存对象
HSET user:1 name "饺子" age 18
HGETALL user:1

# Set：无序、去重，支持交并差
SADD post:1:tags "Redis" "缓存"
SMEMBERS post:1:tags

# ZSet：带分数排序
ZADD rank:score 100 "玩家A" 90 "玩家B"
ZREVRANGE rank:score 0 2 WITHSCORES
```

## 过期与淘汰

- `EXPIRE key 60` / `TTL key`：给键设置存活时间，适合缓存数据。
- 内存不是无限的：配置 `maxmemory` 后，需要选择淘汰策略，如 `allkeys-lru`（所有键中淘汰最近最少使用的）、`volatile-lru`（只在设置了过期时间的键中淘汰）。
- 生产环境一般把 Redis 定位成**缓存**而不是唯一数据源，重要数据仍要落库。

## 持久化

- **RDB**：按时间间隔把内存快照写入磁盘，文件小、恢复快，但可能丢失最近一段数据。
- **AOF**：把写命令追加到日志文件，数据更完整，文件更大、恢复更慢。
- 常见做法是 RDB + AOF 混合使用（Redis 4.0 起支持混合持久化）。

## 典型使用场景

- **缓存**：挡住热点读请求，降低数据库压力。
- **会话存储**：把 Session 放到 Redis，多实例共享。
- **排行榜 / 计数器**：ZSet 和 `INCR` 天然适合。
- **分布式锁**：`SET key value NX EX 30`，注意释放时校验 value 并原子删除。
- **限流**：固定窗口或滑动窗口计数。
- **消息队列**：List 做简单队列，Stream 支持消费组。

## 缓存常见问题

- **穿透**：查询根本不存在的数据，请求每次都落到数据库。可用布隆过滤器或缓存空值缓解。
- **击穿**：某个热点键失效的瞬间，大量请求同时打到数据库。可用互斥锁重建或逻辑过期缓解。
- **雪崩**：大批键在同一时刻失效。可给过期时间加随机值、做多级缓存或提前预热。

## 快速上手

本地最简单的方式是用 Docker 起一个实例：

```bash
docker run -d --name redis -p 6379:6379 redis:7
docker exec -it redis redis-cli

127.0.0.1:6379> SET hello world
OK
127.0.0.1:6379> GET hello
"world"
```

图形化客户端可以用 RedisInsight 或 Another Redis Desktop Manager。

## 小结

Redis 的入门门槛不高：先掌握五种数据类型和对应命令，再理解过期、淘汰、持久化这几个影响可靠性的机制，最后结合“缓存”这个主要定位去了解穿透、击穿、雪崩的处理方式，基本就能在项目里用起来了。
