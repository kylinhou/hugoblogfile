---
title: redis常用命令
categories: ["redis"]
tags: ["redis"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---

查看redis的状态，TPS 控制在6w之内
redis-cli -P 1499 --stat

redis-cli -h 172.25.1.20 -p 7000 --stat
./redis-cli -h 192.168.2.237 -p 6379 --stat


查看内存
```
# Memory


used_memory:13490096 //数据占用了多少内存（字节）

used_memory_human:12.87M //数据占用了多少内存（带单位的，可读性好）

used_memory_rss:13490096  //redis占用了多少内存

used_memory_peak:15301192 //占用内存的峰值（字节）

used_memory_peak_human:14.59M //占用内存的峰值（带单位的，可读性好）

used_memory_lua:31744  //lua引擎所占用的内存大小（字节）

mem_fragmentation_ratio:1.00  //内存碎片率

mem_allocator:libc //redis内存分配器版本，在编译时指定的。有libc、jemalloc、tcmalloc这3种。

如果一个项目的数据量比较大，就要经常用info来看内存的使用量，这样才能让项目更稳定
```
