---
title: 查看rpc是否已开启
categories: ["性能测试"]
tags: ["性能测试", "linux", "rpc"]
date: 2016-06-23
author: "kylin"
grammar_cjkRuby: true
---

# 查看rpc是否已开启
在使用loadrunner进行性能测试的过程中，我们可以监控服务器的资源使用情况，那么需要服务器开启rpc功能。下面简单介绍一下开启rpc的方法。

<!--more-->

### 查看rpc是否已开启
如果使用loadrunner监控服务器资源的时候发现如下的错误信息：

```
Monitor name :UNIX Resources. Internal rpc error (error code:2). Machine: 192.168.2.183. Hint: Check that RPC on this machine is up and running. Check that rstat daemon on this machine is up and running (use rpcinfo utility for this verification). Details: RPC: RPC call failed.
RPC-TCP: recv()/recvfrom() failed.
```

我们先检查一下服务器的rpc功能是不是已经打开了：

```
rpcinfo -p
```

如果如下图所示：

![](http://upload-images.jianshu.io/upload_images/2936641-7721b4dead7a6150.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

说明rpc已经在运行，没有的话我们开启rpc功能

```
rpc.rstatd
```

而如果在执行  之后，提示：-bash: rpcinfo: command not found，说明系统还未安装rpc。那么我们先来安装rpc。

### 安装rpc

一步步按照下面的步骤进行就可以成功的安装：

```
yum install xinetd rsh-server rsh
```

```
wget http://nchc.dl.sourceforge.net/project/rstatd/rstatd/4.0.1/rpc.rstatd-4.0.1.tar.gz
```

```
tar -zxvf rpc.rstatd-4.0.1.tar.gz
```

```
cd rpc.rstatd-4.0.1
```

```
./configure
```

```
make
```


```
make install
```

修改/etc/xinetd.d目录下面的3个conf(rogin,rsh,rexec)中的disable均设置为no

```
cd /etc/xinetd.d
vi  rlogin  　--编辑disable=no，保存
vi  rsh     　--编辑disable=no，保存
vi  rexec     --编辑disable=no，保存
rpc.rstatd    --启动rpc.rstatd进程
```

如果报下面的错误：

```
Cannot register service: RPC: Unable to receive; errno = Connection refused
```

说明还需少一个东西，RPC需要rpcbind，这里要注意看一下你系统的信息，centos6之后改为rpcbind，6之前是叫portmap，安装

```
yum install rpcbind
```

启动

```
rpc.rstatd
```

到这里应该不会报错了，说明成功开启rpc，看一下它的运行状态，如果还是报错：

>Cannot register service: RPC: Unable to receive; errno = Connection refused

可能是rpcbind没有开启，先开启：

```
/etc/init.d/rpcbind start
```

再启动：

```
rpc.rstatd
```

```
rpcinfo -p
```

![](http://upload-images.jianshu.io/upload_images/2936641-7721b4dead7a6150.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



