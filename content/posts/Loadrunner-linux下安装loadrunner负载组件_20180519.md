---
title: linux下安装loadrunner负载组件 
categories: ["性能测试"]
tags: ["性能测试", "linux", "loadrunner"]
date: 2016-08-23
author: "kylin"
grammar_cjkRuby: true
---

# linux下安装loadrunner负载组件

有时候我们的windows压测机的性能不够，需要使用linux服务器充当负载机，下面说一下linux上安装loadrunner负载组件的整个过程。

<!--more-->

### 下载安装包

### 检查防火墙

既然使用该主机做负载，需要关闭它的防火墙，查看以下防火墙的状态：

```
service iptables status
```

如果提示：Firewall is not running，说明防火墙已经关闭，否则将其关闭。

### 安装依赖组件

解压缩安装包之后发现里面有一个compat-libstdc++-33-3.2.3-61.i386.rpm文件，需要先安装这个文件，开始安装

```
rpm -ivh compat-libstdc++-33-3.2.3-61.i386.rpm
```

可能会报如下错误：

>warning: compat-libstdc++-33-3.2.3-61.i386.rpm: Header V3 DSA/SHA1 Signature, key ID e8562897: NOKEY
>error: Failed dependencies:
>        libgcc_s.so.1 is needed by compat-libstdc++-33-3.2.3-61.i386
>        libgcc_s.so.1(GCC_3.0) is needed by compat-libstdc++-33-3.2.3-61.i386
>        libgcc_s.so.1(GCC_3.3) is needed by compat-libstdc++-33-3.2.3-61.i386
>        libgcc_s.so.1(GLIBC_2.0) is needed by compat-libstdc++-33-3.2.3-61.i386

说明缺少依赖组件，我们一点点来安装：

``` 
yum install libgcc.i686
```

```
yun install libgcc.x86_64
```

一般安装完上面两个就可以了，如果还有缺少的组件根据提示继续安装

### 安装loadrunner

先解压文件

```
unzip Linux.zip
```

进入Linux文件夹，运行安装程序installer.sh**

```
cd Linux
sh installer.sh
```

这个时候会报错，提示一些脚本文件权限不足，为了方便起见，一次更改Linux下所有文件的权限

```
cd ..
chmod 777 -R Linux/
```

再次进入Linux目录，运行安装脚本

成功开始安装，根据提示依次输入n（next） a（agree） i （）f （finish）就ok了

### 配置

增加一个LR负载端的客户higkoo

```
useradd -g 0 -s /bin/csh higkoo
```

#### 修改部分LR的配置

```
cd /opt/HP/HP_LoadGenerator/
more env.csh
```

![](http://s13.sinaimg.cn/mw690/9aa583cfgx6C9JejHc07c&690)

```
vi /etc/csh.cshrc
```

在文件的最后一行加上

```
source /opt/HP/HP_LoadGenerator/env.csh
```

切换到higkoo用户，验证一下程序是否安装成功。

```
su higkoo
env
```

![](http://s9.sinaimg.cn/mw690/9aa583cfgx6C9JFsCuY68&690)

```
cd /opt/HP/HP_LoadGenerator/bin
./verify_generator
```

此时出现提示页面如下：

![](http://s12.sinaimg.cn/mw690/9aa583cfgx6C9JRANf53b&690)

要求我们设置一个DISPLAY 变量，我们直接修改env.csh文件来解决。

先切换到root用户下，因为两个原因：

1、env.csh文件对higkoo用户是只读的，无法修改

2、修改env.csh用户，相当于修改csh的配置，这个时候需要重新进入一下csh，才能生效

所以我们先退回到root下，修改env.csh 文件。

```
exit   ////切换回root用户
vi /opt/HP/HP_LoadGenerator/env.csh
```

直接在文件最后方加上 

```
setenv DISPLAY 0.0
```

切换到higkoo用户，再次运行验证程序

```
su higkoo
cd /opt/HP/HP_LoadGenerator/bin
./verify_generator
```

![](http://s10.sinaimg.cn/mw690/9aa583cfgx6C9LSuljHe9&690)

#### 启动LR 负载端

```
cd /opt/HP/HP_LoadGenerator/bin
./m_daemon_setup start
```

如果能跟正常启动非常幸运，如果不行的话一般需要进行如下的操作：

查看当前shell下的主机名：

```
env
```

会看到这样的信息：

>[root@iZrj9ficl3p7xemrfle8zgZ ~]# env
>HOSTNAME=**iZrj9ficl3p7xemrfle8zgZ**
>TERM=xterm
>SHELL=/bin/bash
>HISTSIZE=1000

注意上面的HOSTNAME的值，这就是主机名，我们ping一下看看能不能通

```
ping iZrj9ficl3p7xemrfle8zgZ
```

会发现连接不上，loadrunner现在无法启动的原因就在这里。

切换到root下，更改hosts配置，把bogon指向127.0.0.1

```
exit
vi /etc/hosts
```

> 127.0.0.1 localhost iZrj9ficl3p7xemrfle8zgZ
> ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

按照上面的格式将主机名绑定到127.0.0.1上

再ping一下：

```
ping iZrj9ficl3p7xemrfle8zgZ
```

将会发现能够ping通

下面重新启动loadrunner

```
cd /opt/HP/HP_LoadGenerator/bin
./m_daemon_setup start
```

如果输出类似下面的信息，说明成功启动

> **m_agent_daemon ( 26299 )**

### 检查loadrunner是否正常启动

```
ps aux | grep m_agent_daemon
```

你将会看到下面的信息：

> [root@iZrj9ficl3p7xemrfle8zgZ ~]# ps aux | grep m_agent_daemon
> higkoo   26299 0.1  4.4 737356 727868 ?       S   (后面的内容略掉)
>
> root     17552  0.0  0.0 103260   840 pts/0    S+   17:21   0:00 grep m_agent_daemon

