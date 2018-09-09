---
title: redis使用pipeline批量造数据
categories: ["redis"]
tags: ["redis", "造数据"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---
# redis使用pipeline批量造数据

[Redis](http://lib.csdn.net/base/redis)客户端与[redis](http://lib.csdn.net/base/redis)之间使用TCP协议进行连接，一个客户端可以通过一个socket连接发起多个请求命令。每个请求命令发出后client通常会阻塞并等待redis服务处理，redis处理完后请求命令后会将结果通过响应报文返回给client,因此当执行多条命令的时候都需要等待上一条命令执行完毕才能执行 

而管道（pipeline）可以一次性发送多条命令并在执行完后一次性将结果返回，pipeline通过减少客户端与redis的通信次数来实现降低往返延时时间 

普通方式与pipeline方式插入数据的性能对比可以参考这篇[文章](https://blog.csdn.net/xiangxizhishi/article/details/74275755) 

