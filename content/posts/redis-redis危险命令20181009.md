---
title: redis危险命令
categories: ["redis"]
tags: ["redis"]
date: 2018-10-09
author: "kylin"
grammar_cjkRuby: true
---
# redis危险命令

## keys

虽然keys的模糊匹配功能使用非常方便也很强大，在小数据量情况下使用没什么问题，数据量大会导致 Redis 锁住及 CPU 飙升，在生产环境建议禁用或者重命名 。

## flushdb

删除 Redis 中所有数据库中的所有记录，不只是当前所在数据库，并且此命令从不会执行失败 

## config

客户端可修改 Redis 配置 

## 禁用或重命名命令

#### 怎么禁用或重命名危险命令？

看下 `redis.conf` 默认配置文件，找到 `SECURITY` 区域，如以下所示。

```
################################## SECURITY ###################################



# Require clients to issue AUTH <PASSWORD> before processing any other

# commands.  This might be useful in environments in which you do not trust

# others with access to the host running redis-server.

#

# This should stay commented out for backward compatibility and because most

# people do not need auth (e.g. they run their own servers).

#

# Warning: since Redis is pretty fast an outside user can try up to

# 150k passwords per second against a good box. This means that you should

# use a very strong password otherwise it will be very easy to break.

#

# requirepass foobared



# Command renaming.

#

# It is possible to change the name of dangerous commands in a shared

# environment. For instance the CONFIG command may be renamed into something

# hard to guess so that it will still be available for internal-use tools

# but not available for general clients.

#

# Example:

#

# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52

#

# It is also possible to completely kill a command by renaming it into

# an empty string:

#

# rename-command CONFIG ""

#

# Please note that changing the name of commands that are logged into the

# AOF file or transmitted to slaves may cause problems.
```

看说明，添加 `rename-command` 配置即可达到安全目的。

**1）禁用命令**

```
rename-command KEYS     ""

rename-command FLUSHALL ""

rename-command FLUSHDB  ""

rename-command CONFIG   ""
```

**2）重命名命令**

```
rename-command KEYS     "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

rename-command FLUSHALL "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

rename-command FLUSHDB  "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

rename-command CONFIG   "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```

上面的 XX 可以定义新命令名称，或者用随机字符代替。

参考自文章：http://www.imooc.com/article/247139#

