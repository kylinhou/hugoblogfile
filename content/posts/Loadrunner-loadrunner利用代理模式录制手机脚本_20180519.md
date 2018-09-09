---
title: loadrunner利用代理模式录制手机脚本
categories: ["性能测试"]
tags: ["性能测试", "loadrunner", "hexo"]
date: 2016-08-26
author: "kylin"
grammar_cjkRuby: true
---

# loadrunner利用代理模式录制手机脚本

loadrunner12有手机版，可以直接在手机上录制脚本，但是需要root手机

loadrunner11和loadrunner12电脑版也是可以录制手机脚本的，设置一下代理就可以。

<!--more-->

#### loadrunner的设置

下面是loadrunner11的方法

首先创建脚本，选择WEB(HTTP/HTML)协议

弹出录制选项框，这里需要设置一下，如下图所示：

![120101.png](http://upload-images.jianshu.io/upload_images/2936641-52ec26644766db5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里注意program to record选项要重新设置一下，选择loadrunner安装目录下的bin目录下的wplus_init_wsock.exe

url可以不用写

然后点击options选项，如下图所示：

![120102.png](http://upload-images.jianshu.io/upload_images/2936641-0f2c82bb7d26de84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择network下的port mapping选项，然后点击new entry，进入如下界面：

![120103.png](http://upload-images.jianshu.io/upload_images/2936641-dc0f03297e1a26d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

target server：要录制的脚本所访问的网站域名，如果访问的地址跟这里填写的不一样手机将会访问不了。

port：80

service ID：HTTP

traffic forwarding：选中，然后填写一个端口号，这个端口号是给手机做代理用的。注意使用的端口号是你电脑空闲的一个端口号

设置这些就可以了，填好之后点击update

#### 手机端的设置

手机上需要设置代理，让手机的流量通过你loadrunner所在的电脑，原理还是让loadrunner充当一个代理服务器。实现这个的话需要下面两个条件：

1. 手机所连接的wifi与电脑在同一局域网
2. 手机连接wifi之后设置代理模式。

如果是笔记本的话第一个条件很好满足，只需要开一个热点让手机连接就行。台式机的话就需要随身wifi这样的外部设备。

手机设置代理模式：

![](http://img.blog.csdn.net/20150817100515342?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



注意这里，主机名就是电脑的ip，端口号是traffic forwarding那里设置的端口号。



经过上面的设置，就可以开始录制了。电脑上点击录制开始按钮，可以看到启动了代理服务器，然后在手机上进行操作就可以了。注意loadrunner的信息提示，如果有什么问题根据信息做更改。



