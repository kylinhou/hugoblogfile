---
title: 监控redis
categories: ["redis"]
tags: ["性能测试", "redis"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---
# 监控redis

## 监控执行的redis指令

redis-cli monitor | grep '\[0' | grep -v 'PING' | grep -v 'SELECT'

redis-cli -h 192.168.2.231 -p 1307 monitor | grep '\[0' | grep -v 'PING' | grep -v 'SELECT'

-v ：过滤掉某些词
