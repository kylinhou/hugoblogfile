---
title: redis的安装与配置
categories: ["redis"]
tags: ["redis"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---
# redis安装与配置

## redis3.0.7安装
1：上传安装文件到服务器
![upload](images\upload.png)

解压：
> tar -zxvf redis-3.0.7.tar.gz

进行编译安装

>cd /home/soft/redis3.0.7
>make
>make install

如果出现如下问题：
![pro](images\pro.png)

先执行一下 make distclean
然后make、make install

不出意外的话就安装成功了。

执行redis-server命令：
![成功](images\成功.png)

可以看到安装成功。

## 创建节点并指定端口
同一台服务器上可以同时让多个项目共用这个redis，每个项目使用不同的端口号就行。
创建一个文件夹：
> mkdir redis-theme-1003

将redis的配置文件复制到这个目录下：
> cp redis-3.0.7/redis.conf redis-theme-1003

修改redis-theme-1003下的配置文件：
* 修改端口号
   port 7000 将端口号改成自己需要的端口号，例如本次1000 或者1001

* redis作为守护进程运行
  daemonize yes

* tcp-keepalive值设置为60s

## 运行redis
需要使用root账户才能启动，具体原因不明
> redis-server ./redis.conf &

查看是否启动成功
> ps -ef | grep redis
> ![qid](images\qid.png)

## redis安装报错

###  Test replication partial resync: ok psync (diskless: yes, reconnect: 1) in tests/integration/replication-psync.tcl

Expected condition '[s -1 sync_partial_ok] > 0' to be true ([s -1 sync_partial_ok] > 0)

**1，只用单核运行 make test：**

> **taskset -c 1 sudo make test**

**2，更改 tests/integration/replication-psync.tcl 文件：**

> **vi tests/integration/replication-psync.tcl**

把对应报错的那段代码中的 **after**后面的数字，从100改成 **500**。我个人觉得，这个参数貌似是等待的毫秒数。

## 安装tcl

1. wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz           //直接下载 
2. sudo tar xzvf tcl8.6.1-src.tar.gz  -C /usr/local/  
3. cd  /usr/local/tcl8.6.1/unix/  
4. sudo ./configure  
5. sudo make  
6. sudo make install 

## 安装gcc 

 > yum install gcc 





