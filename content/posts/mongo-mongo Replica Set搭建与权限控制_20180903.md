---
title: mongo Replica Set搭建与权限控制
categories: ["mongo"]
tags: ["编程", "mongo"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---

# mongo Replica Set搭建与权限控制

## mongo集群说明

mongodb的集群搭建方式主要有三种，主从模式，Replica set模式，sharding模式, 三种模式各有优劣，适用于不同的场合，属Replica set应用最为广泛，主从模式现在用的较少，sharding模式最为完备，但配置维护较为复杂。这里主要介绍Replica Set模式的搭建方法。

使用mongo版本：3.2.9

## mongo下载

以3.2.9版本为例：

wget --no-check-certificate  https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-amazon-3.2.9.tgz

## 单节点安装与配置

将下载的文件解压到mongo的安装路径

本文的mongo安装路径：/home/mongo/mongodb-linux-x86_64-amazon-3.2.9

tar vzxf mongodb-linux-x86_64-amazon-3.2.9.tgz

把mongo添加到环境变量里面：

vi /etc/profile.d/mongodb.sh

文件内容：

```
export MONGO_HOME=/home/mongo/mongodb-linux-x86_64-amazon-3.2.9
export PATH=$PATH:$MONGO_HOME/bin
```

然后  source /etc/profile.d/mongodb.sh

检验环境变量是否生效：

mongod --version

## mongo **Replica Set**搭建

搭建一个 Replica Set mongo集群，使用3台服务器

172.25.34.4

172.25.34.5

172.25.34.6

端口号统一使用27011

在三台服务器上按照上一步的说明安装完mongo，分别在三台服务器上进行如下操作：

### 1.创建一个目录保存我们mongo集群的信息

> mkdir appstore-mongo-27011

集群的根目录就是：/home/mongo/appstore-mongo-27011

> cd appstore-mongo-27011

### 2.创建3个目录，分别来保存日志、数据、keyfile

>  mkdir log db keyfile

### 3.创建一个 mongodb.conf 文件

创建一个 mongodb.conf 文件作为我们集群的配置文件

> vim mongodb.conf 

>dbpath=/home/mongo/appstore-mongo-27011/db
>logpath=/home/mongo/appstore-mongo-27011/log/mongod.log
>pidfilepath=/home/mongo/appstore-mongo-27011/mongo.pid
>directoryperdb=true
>logappend=true
>
>replSet=appstoremongo
>
>bind_ip=172.25.34.5
>port=27011
>oplogSize=10000
>fork=true
>noprealloc=true
>storageEngine=wiredTiger

注意：

* replSet：指定一个副本集名称作为参数，所有主机都必须有相同的名称作为同一个副本集。-- 集群配置项

* 注意修改ip、端口号、路径

* logpath必须具体到文件，如果只是到文件夹，mongo将无法启动

  这一步结束之后，集群根目录下的文件结构如下：

  > [mongo@localhost appstore-mongo-27011]$ pwd
  > /home/mongo/appstore-mongo-27011
  > [mongo@localhost appstore-mongo-27011]$ ll
  > total 20
  > drwxrwxr-x 7 mongo mongo 4096 Mar  3 16:12 db
  > drwxrwxr-x 2 mongo mongo 4096 Mar  3 16:07 keyfile
  > drwxrwxr-x 2 mongo mongo 4096 Mar  3 15:58 log
  > -rw-rw-r-- 1 mongo mongo  431 Mar  3 16:08 mongodb.conf
  > -rw-rw-r-- 1 mongo mongo    7 Mar  3 16:08 mongo.pid

### 4.启动各个节点

mongod -f /home/mongo/appstore-mongo-27011/mongodb.conf

注：如果未将mongo添加到环境变量无法直接使用mongod

### 5.创建副本集

登陆mongo

> mongo 172.25.34.4:27011

切换到admin

> use admin

提交副本集信息

```shell
cfg={ _id:"appstoremongo", members:[ {_id:0,host:'172.25.34.4:27011',priority:2}, {_id:1,host:'172.25.34.5:27011',priority:1}, {_id:2,host:'172.25.34.6:27011',arbiterOnly:true}] }; 
```

注：_id:"appstoremongo" 必须与配置文件中副本集名称一致

priority表示优先级别，数值越大，表示是主节点

arbiterOnly:true表示仲裁节点

### 6.初始化副本集

> rs.initiate(cfg) 

  执行成功后，出现如下信息

​	{ "ok" : 1 }

   可以执行rs.status()来检查执行配置结果信息

### 7.添加超级用户

登录到服务器的admin库

> use admin

创建超级用户

```shell
> \>db.createUser({user: "root",pwd: "root",roles: [ { role: "root", db:"admin" }]})
```

后面修改配置文件后，需要用这个超级用户来登录并创建数据库及用户名和密码，否则无法登录新建用户等操作。

所以一定要在开启权限认证之前创建超级用户

## 权限管理配置

在开启权限认证之前，一定要完成超级用户的创建！

###  关闭各服务器

```shell
mongod --shutdown  -f ./mongodb.conf
```

### 生成keyfile

```shell
openssl rand -base64 741 > /home/mongo/appstore-mongo-27011/keyfile/mongodb-keyfile
```

### 复制keyfile到另外两台服务器

> scp ./mongodb-keyfile user@ip:path

### 修改keyfile权限

```shell
chmod 600 ./mongodb-keyfile
```

注：一定要600

### 修改配置文件

配置文件中添加如下两项：

> auth=true
> keyFile=/home/mongo/appstore-mongo-27011/keyfile/mongodb-keyfile

```
> auth=true
> keyFile=/home/mongo/appstore-mongo-27011/keyfile/mongodb-keyfile
```

### 重启各服务器

> mongod -f ./mongodb.conf

### 登录并创建用户

先用root用户登陆admin

```sh
use admin
```

```sh
db.auth("root","root")
```

mongod的用户权限是跟着数据库走的，所以在创建之前先进入对应的db

下面的命令相当于创建数据库，并设置角色与权限

> use appstore

创建appstore用户，拥有appstore db的读写权限

```shell
db.createUser({user:'appstore',pwd:'appstore',roles:[{role:'readWrite',db:'appstore'}]})
```

ok了，到此结束。可以使用客户端登陆验证。

## 关于配置文件

详细请参考官方文档：

<http://docs.mongoing.com/manual-zh/reference/configuration-options.html>

* dbpath=/opt/data/mongo/    

  数据存放目录

* logpath=/opt/data/mongo/log/mongo.log  

  日志路径，注意路径到文件，否则启动不成功

* pidfilepath=/opt/data/mongo/mongo.pid 

  进程ID，没有指定则启动时候就没有PID文件。默认缺省。

* directoryperdb=true 

  设置为true，修改数据目录存储模式，每个数据库的文件存储在DBPATH指定目录的不同的文件夹中。使用此选项，可以配置的MongoDB将数据存储在不同的磁盘设备上，以提高写入吞吐量或磁盘容量。默认为false。

  在一开始就要确定好该参数值，因为中途修改改参数值，会因为数据目录的改变导致数据可能不可用，注意！

* logappend=true

  写日志的模式：设置为true为追加。默认是覆盖。如果未指定此设置，启动时MongoDB的将覆盖现有的日志文件。

* bind_ip=192.168.2.237

  绑定地址。默认127.0.0.1，只能通过本地连接。进程绑定和监听来自这个地址上的应用连接。要是需要给其他服务器连接，则需要注释掉这个或则把IP改成本机地址，如192.168.200.201[其他服务器用 mongo--host=192.168.200.201 连接] ，可以用一个逗号分隔的列表绑定多个IP地址。

* port=10000

  默认27017，MongoDB的默认服务TCP端口，监听客户端连接。要是端口设置小于1024，比如1021，则需要root权限启动。

* storageEngine=wiredTiger

  存储引擎类型，mongodb 3.0之后支持“mmapv1”、“wiredTiger”两种引擎，默认值为“mmapv1”；官方宣称wiredTiger引擎更加优秀。

* noprealloc=true

  默认false：使用预分配方式来保证写入性能的稳定，预分配在后台进行，并且每个预分配的文件都用0进行填充。这会让MongoDB始终保持额外的空间和空余的数据文件，从而避免了数据增长过快而带来的分配磁盘空间引起的阻塞。
  设置noprealloc= true来禁用预分配的数据文件，会缩短启动时间，但在正常操作过程中，可能会导致性能显著下降。

* oplogSize=10000

  指定的复制操作日志（OPLOG）的最大大小。mongod创建一个OPLOG的大小基于最大可用空间量。对于64位系统，OPLOG通常是5％的可用磁盘空间。一旦mongod第一次创建OPLOG，改变oplogSize将不会影响OPLOG的大小。

* fork=true

  是否后台运行，设置为true 启动 进程在后台运行的守护进程模式。默认false。

* auth=true

  用户认证，默认false。不需要认证。当设置为true时候，进入数据库需要auth验证，当数据库里没有用户，则不需要验证也可以操作。直到创建了第一个用户，之后操作都需要验证。

* keyFile=/opt/data/mongo/keyfile/mongo_keyfile

  指定存储身份验证信息的密钥文件的路径。默认缺省。（分片或者副本集创建用户的时候，共享身份验证秘钥—多台机器互相信任彼此的“通信证”）

* replSet=testrs

  指定一个副本集名称作为参数，所有主机都必须有相同的名称作为同一个副本集。-- 集群配置项

* master = true

  master：默认为false，当设置为true，则配置当前实例作为主实例。-- 主从配置的主库配置项

* slave = true

  默认为false，当设置为true，则配置当前实例作为从实例。 -- 主从配置的丛库配置项

* source：默认为空，格式为：<host><:port>。用于从实例的复制：设置从的时候指定该选项会让从复制指定主的实例。 --主从配置的丛库配置项

* only = abc        

  默认为空，则会同步所有库，如果不为空，则同步指定的那个库。--主从配置的丛库配置项

* slavedelay = 60     #延迟60s同步主数据

  设置从库同步主库的延迟时间，用于从设置，默认为0。--主从配置的丛库配置项

#### 一份配置文件：

```sh
#master.conf
setParameter:
   enableLocalhostAuthBypass: false

systemLog:
   verbosity: 0 # 日志级别。默认0
   quiet: false  # 设置为true attempts to limit the amount of output.  systemLog.quiet is not recommended for production systems as it may make tracking problems during particular connections much more difficult.

   traceAllExceptions: false  # 调试用
   path: /data/mongodb/logs/mongo.log
   logAppend: true
   logRotate: reopen       #日志文件重写方式
   destination: file
   timeStampFormat: iso8601-local        #时间格式  iso8601-local iso8601-utc  ctime

processManagement:
   fork: true
   pidFilePath: /data/mongodb/tmp/mongo.pid

net:
   port: 27017
   maxIncomingConnections: 65536  # 默认值
   wireObjectCheck: true         #对文档合法性进行检查
   unixDomainSocket:             # 监听socket 端口
      enabled: true
      pathPrefix: /data/mongodb/tmp/
      filePermissions: 0

storage:
   dbPath: /data/mongodb/data/
   indexBuildRetry: true   # 每个索引单独一个文件
   journal:
      enabled: true
      commitIntervalMs: 100  # 处理journal 日志的时间间隔
   directoryPerDB: true
   syncPeriodSecs: 60        # 没60s 写一次磁盘 对 journal files  无作用
   engine: wiredTiger
   wiredTiger:
      engineConfig:
         cacheSizeGB: 32

replication:
   replSetName: rs0
   oplogSizeMB: 50000  ##50G
security:
   authorization: enabled
   clusterAuthMode: keyFile
   keyFile: /data/mongodb/keyfile
```

## 用户权限说明

mongo中的角色：

```
Read：允许用户读取指定数据库
readWrite：允许用户读写指定数据库
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
root：只在admin数据库中可用。超级账号，超级权限
```
## 遇到的问题

### 无主节点

1：在初始化副本之后，执行rs.status() 查看集群的状态，发现3个节点没有一个节点是主节点，有两个节点一直是"stateStr" : "STARTUP"状态，原因是服务器磁盘没有空间了，查看下磁盘

2：如果第一步没有成功，继续在其他节点初始化看看，如果报错：

> replSetInitiate quorum check failed because not all proposed set members responded affirmatively: 172.25.1.32:27012 failed with No route to host

是因为mongodb分布式复制集初始化时需要各个节点之间的通话，所以需要将各个节点上的防火墙进行关闭 

方法一：命令为：# service iptables stop   （即时生效，临时关闭防火墙，重系统后防火墙会自动开启）

方法二：# chkconfig iptables off  （重启后生效，永久关闭防火墙）



