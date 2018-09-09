---
title: linux下安装nload
categories: ["linux"]
tags: ["性能测试", "linux", "nload"]
date: 2016-10-04
author: "kylin"
grammar_cjkRuby: true
---

# linux下安装nload

## centos下安装nload监控网卡的流量情况

#### 1、安装依赖包

```
yum install -y gcc gcc-c++ ncurses-devel make wget
```

<!--more-->

#### 2、下载Nload

```
wget http://www.roland-riegel.de/nload/nload-0.7.4.tar.gz
```

#### 3、安装Nload

```
tar zxvf nload-0.7.4.tar.gz
cd nload-0.7.4
./configure
make
make install
```

#### 4、运行Nload

```
nload
```



## nload的基本使用

直接输入nload就可以使用，不过这样显示的单位是Mbit/s

![nload](http://upload-images.jianshu.io/upload_images/2936641-6c734aabe9315d76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果想看的方便一点可以使用

```
nload -u M
```

这样显示的单位就是MByte/s，也就是平时我们说的兆/每秒

下面说一下每一项的含义：

> 默认第一行是网卡的名称及IP信息，使用键盘上的左右键可以切换网卡。
> 默认上边Incoming是进入网卡的流量;
> 默认下边Outgoing是网卡出去的流量;
> 默认右边（Curr当前流量）、（Avg平均流量）、（Min最小流量）、（Max最大流量）、（Ttl流量统计）;
> 默认情况，统计数据的左边会使用显示流量图，用#号拼出来的，根据实时流量变化显示。
> -a：这个好像是全部数据的刷新时间周期，单位是秒，默认是300.
> -i：进入网卡的流量图的显示比例最大值设置，默认10240 kBit/s.
> -m：不显示流量图，只显示统计数据。
> -o：出去网卡的流量图的显示比例最大值设置，默认10240 kBit/s.
> -t：显示数据的刷新时间间隔，单位是毫秒，默认500。
> -u：设置右边Curr、Avg、Min、Max的数据单位，默认是自动变的.注意大小写单位不同！





