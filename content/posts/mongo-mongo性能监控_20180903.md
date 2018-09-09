---
title: mongo性能监控
categories: ["mongo"]
tags: ["编程", "mongo"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---

# mongo性能监控

2018/4/13

侯兴隆

## 一：mongostat详解

mongostat是mongdb自带的状态检测工具，在命令行下使用。它会间隔固定时间获取mongodb的当前运行状态，并输出。

基本的使用方法：

> ./mongostat --host 172.25.34.8 --port 27012 -uroot -proot --authenticationDatabase admin

如果需要用户名和密码登录就像上面一样，修改下自己的用户名、密码、认证库就可以了。

执行命令之后可以看到：

> [mongo@localhost bin]$ ./mongostat --host 172.25.34.8 --port 27012 -uroot -proot --authenticationDatabase admin
> insert query update delete getmore command % dirty % used flushes vsize   res qr|qw ar|aw netIn netOut conn        set repl                      time
>     *0    *0     *0     *0       0     4|0     0.0   31.9       0 12.6G 11.7G   0|0   0|0  469b  21.0k   56 musicmongo  PRI 2018-04-13T19:49:45+08:00
>     *0    *0     *0     *0       0     1|0     0.0   31.9       0 12.6G 11.7G   0|0   0|0   79b  19.7k   56 musicmongo  PRI 2018-04-13T19:49:46+08:00
>     *0    *0     *0     *0       0     3|0     0.0   31.9       0 12.6G 11.7G   0|0   0|0  712b  20.2k   56 musicmongo  PRI 2018-04-13T19:49:47+08:00

它的输出有以下几列：

- inserts/s 每秒插入次数
- query/s 每秒查询次数
- update/s 每秒更新次数
- delete/s 每秒删除次数
- getmore/s 每秒执行getmore次数
- command/s 每秒的命令数，比以上插入、查找、更新、删除的综合还多，还统计了别的命令
- flushs/s 每秒执行fsync将数据写入硬盘的次数。
- mapped/s 所有的被mmap的数据量，单位是MB，
- vsize 虚拟内存使用量，单位MB
- res 物理内存使用量，单位MB
- faults/s 每秒访问失败数（只有Linux有），数据被交换出物理内存，放到swap。不要超过100，否则就是机器内存太小，造成频繁swap写入。此时要升级内存或者扩展
- locked % 被锁的时间百分比，尽量控制在50%以下吧
- idx miss % 索引不命中所占百分比。如果太高的话就要考虑索引是不是少了
- q t|r|w 当Mongodb接收到太多的命令而数据库被锁住无法执行完成，它会将命令加入队列。这一栏显示了总共、读、写3个队列的长度，都为0的话表示mongo毫无压力。高并发时，一般队列值会升高。
- conn 当前连接数
- time 时间戳

## 二：使用profiler

类似于MySQL的slow log, MongoDB可以监控所有慢的以及不慢的查询。

### 开启查询记录

Profiler默认是关闭的，你可以选择全部开启，或者有慢查询的时候开启。

> ```
> > use test
> switched to db test
> > db.setProfilingLevel(2);
> {"was" : 0 , "slowms" : 100, "ok" : 1} // "was" is the old setting
> > db.getProfilingLevel()
> 2
> ```

上面profile的级别可以取0，1，2  三个值，他们表示的意义如下： 

0 –  不开启 

1 –  记录慢命令 (默认为>100ms)  

2 –  记录所有命令  

### 查看Profile日志:

> db.system.profile.find().sort({$natural:-1})
> {"ts" : "Thu Jan 29 2009 15:19:32 GMT-0500 (EST)" , "info" :
> "query test.$cmd ntoreturn:1 reslen:66 nscanned:0 query: { profile: 2 } nreturned:1 bytes:50" ,
> "millis" : 0} ...

3个字段的意义

- ts：时间戳
- info：具体的操作
- millis：操作所花时间，毫秒

与 MySQL 的慢查询日志不同，MongoDB Profile  记录是直接存在系统 db 里的，记录位置 system.profile  ，所以，我们只要查询这个 Collection 的记录就可以获取到我们的 Profile  记录了。列出执行时间长于某一限度(5ms)的 Profile  记录： 

> db.system.profile.find( { millis : { $gt : 5 } } ) 

查看最新的 Profile  记录： 
>  db.system.profile.find().sort({$natural:-1}).limit(1) 
>  { "ts" : ISODate("2012-05-20T16:50:36.321Z"), "info" : "query test.system.profile reslen:1219 
>  nscanned:8  \nquery: { query: {}, orderby: { $natural: -1.0 } }  nreturned:8 bytes:1203", "millis" : 
>  0 } 

## 三：使用Web控制台

Mongodb自带了Web控制台，默认和数据服务一同开启。他的端口在Mongodb数据库服务器端口的基础上加1000，如果是默认的Mongodb数据服务端口(Which is 27017)，则相应的Web端口为28017

这个页面可以看到

- 当前Mongodb的所有连接
- 各个数据库和Collection的访问统计，包括：Reads, Writes, Queries, GetMores ,Inserts, Updates, Removes
- 写锁的状态
- 以及日志文件的最后几百行（CentOS+10gen yum 安装的mongodb默认的日志文件位于/var/log/mongo/mongod.log)

## 四：db.currentOp()

Mongodb 的命令一般很快就完成，但是在一台繁忙的机器或者有比较慢的命令时，你可以通过db.currentOp()获取当前正在执行的操作。

在没有负载的机器上，该命令基本上都是返回空的

>&nbsp; db.currentOp()
>{ "inprog" : [ ] }

字段名字都能自解释。如果你发现一个操作太长，把数据库卡死的话，可以用这个命令杀死他

> db.killOp("shard3:466404288")

## 五：使用Web控制台

Mongodb自带了Web控制台，默认和数据服务一同开启。他的端口在Mongodb数据库服务器端口的基础上加1000，如果是默认的Mongodb数据服务端口(Which is 27017)，则相应的Web端口为28017

这个页面可以看到

- 当前Mongodb的所有连接
- 各个数据库和Collection的访问统计，包括：Reads, Writes, Queries, GetMores ,Inserts, Updates, Removes
- 写锁的状态
- 以及日志文件的最后几百行（CentOS+10gen yum 安装的mongodb默认的日志文件位于/var/log/mongo/mongod.log)

新版本的mongo默认关闭了web控制台，可以在启动mongo的时候开启：

> ~$ sudo mongod --httpinterface --dbpath /var/lib/mongodb/ --logpath /var/log/mongodb/mongod.log --logappend &

--httpinterface 就是设定开启web服务的，其他参数是设置数据库存储路径，log日志文件存储路径，以及设置log日志记录方式为添加而不是覆盖原来的。



