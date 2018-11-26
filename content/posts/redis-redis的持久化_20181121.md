---
title: redis的持久化
categories: ["redis"]
tags: ["性能测试", "redis"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---
# redis的持久化

## redis的持久化方式

Redis是一种高级key-value数据库。它跟memcached类似，不过数据可以持久化，而且支持的数据类型很丰富。有字符串，链表，集 合和有序集合。支持在服务器端计算集合的并，交和补集(difference)等，还支持多种排序功能。所以Redis也可以被看成是一个数据结构服务器。
Redis的所有数据都是保存在内存中，然后不定期的通过异步方式保存到磁盘上(这称为“半持久化模式”)；也可以把每一次数据变化都写入到一个append only file(aof)里面(这称为“全持久化模式”)。

下面具体介绍一下这两种方式。

## Redis DataBase(简称RDB)



## Append-only file (简称AOF)

### 什么是AOF持久化

AOF持久化跟RDB持久化不同，并不是把内存数据写入文件，而是把每次的执行命令写入文件，查询操作不会记录。跟mysql的binlog类似。如果开启了AOF持久化，在redis重启时，会执行AOF文件中的命令来恢复数据。

AOF持久化更能达到实时性、完整性。

### 如何开启AOF持久化

打开AOF持久化机制，需要在配置文件中修改

appendonly no -> appendonly yes

### AOF持久化的内部机制

#### 同步机制

Redis提供了多种AOF缓冲区同步文件策略，由参数appendfsync控制

* appendfsync always：每次写入一条数据，立即将这个数据对应的写日志fsync到磁盘上去，fsync完成后线程返回，这种做法虽然确保了数据不会丢失但性能却很差。
* appendfsync everysec：专门的线程每秒将缓存区中的数据fsync到磁盘，这个最常用的，生产环境一般都这么配置，性能较高
* appendfsync no：redis仅仅负责将数据写入缓存区，后续的工作由操作系统完成，操作系统根据自身的策略将数据刷入磁盘，虽可提升性能,但这种做法变得不可控。

#### 触发时机

1. 手动触发

   bgrewriteaof命令:和bgsave有些类似，都是fork子进程进行具体的工作，且都只有在fork时阻塞。

2. 自动触发


### AOF持久化的优缺点



## 参考文章：

 [redis持久化的几种方式](https://www.cnblogs.com/chenliangcl/p/7240350.html)

