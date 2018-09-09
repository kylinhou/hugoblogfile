---
title: Loadrunner参数化使用mysql数据源
categories: ["性能测试"]
tags: ["loadrunner", "性能测试"]
date: 2016-09-02
author: "kylin"
grammar_cjkRuby: true
---

# Loadrunner 参数化使用mysql数据源 

## 问题概述

在使用loadrunner进行性能测试的过程中，参数化是非常重要的一个操作，有时候我们参数化的数据需要跟数据库中的数据一致才能有效的完成测试。我们可以选择将数据库中的对应字段的数据导出到文件，然后把数据copy到脚本文件中，但是还有更简单的方法，我们可以直接使用loadrunner连接我们的数据库，将对应字段的数据取出直接保存到参数化脚本文件中。

<!--more-->

下面介绍使用mysql参数化的一般方法

## 操作方法

### 安装数据库驱动，连接mysql

要想富先修路，当然要先保证能够正常的连接到mysql，不管是本地的还是远程的。

下载这个货：*mysql-connector-odbc-5.3.6-win32*

版本自己可以选择一个合适的，但是一定要用32位的！！！因为loadrunner是32位，不然匹配不了。

### 打开odbc，设置数据源

在电脑开始栏中输入odbc，就可以看到安装好的odbc程序，打开

![](http://img.blog.csdn.net/20140331224056656?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhbmdjaGFveQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后在User DSN中点Add，选择刚添加的mysql驱动

![](http://img.blog.csdn.net/20140331224539296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhbmdjaGFveQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



Data Source Name 和 Description中，随意填写。server中填写服务器的ip

User name 和Password就不用说了吧。Database是选择要使用的数据库。

点test来[测试](http://lib.csdn.net/base/softwaretest)一下。好了， 连接成功。

回到LoadRunner中，在参数化窗口中点Data Wizard...按钮。

![](http://img.blog.csdn.net/20140331225125406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhbmdjaGFveQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![](http://img.blog.csdn.net/20140331225322421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhbmdjaGFveQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里要 选中Specify SQL statement manually.继续next，将会看到下一个界面，在machine Data Source中选中数据源

![](http://img.blog.csdn.net/20140331225402421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhbmdjaGFveQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

点击ok之后

![](http://upload-images.jianshu.io/upload_images/2936641-fc7baea57598ea04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里要注意，数字1的地方是自动填写好的，如果发现这里没有自动生成，说明你之前的操作是有问题的，需要重新来过。

数字2的地方是我们的sql语句，假设我需要取出lt_test表中的name字段，我们就可以select name from lr_test，这很容易理解。

好了，到这里之后就全部完成了，点击finish，就会看到数据已经成功从数据库中取出。



## 注意事项

### 找不到mysql驱动

在上面的进行过程中，我们打开odbc，在User DSN中点Add，结果找不到mysql的驱动

![](http://upload-images.jianshu.io/upload_images/2936641-966a312d7c94d98f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

啊，没有mysql的啊啊 啊，徒儿莫慌，为师自有妙计

打开 *C:\Windows\SysWOW64*  文件夹，找到 **odbcad32** 程序

![](http://upload-images.jianshu.io/upload_images/2936641-eb8dcfac753a4d39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开，在这里你就会看到好多的程序啊 啊，安装的mysql的驱动程序也在里面，找到打开，就跟上面的操作步骤一样了。

![](http://upload-images.jianshu.io/upload_images/2936641-20b81a060ed6150b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

