---
title: loadrunner学习
categories: ["性能测试"]
tags: ["loadrunner", "性能测试"]
date: 2016-06-23
author: "kylin"
grammar_cjkRuby: true
---

# loadrunner学习

## loadrunner介绍

**LoadRunner **是 HP Mercury Interactive用来测试应用程序性能的工具

LoadRunner 通过模拟一个多用户并行工作的环境来对应用程序进行负载测试。通过使用最少的硬件资源，这些虚拟用户提供一致的、可重复并可度量的负载，像实际用户一样使用所要测试的应用程序。LoadRunner 深入的报告和图提供了评估应用程序性能所需的信息。 

<!--more-->

### loadrunner测试过程

* 制定负载测试计划 
* 开发测试脚本
* 创建运行场景 
* 执行测试
* 监视场景 
* 分析测试结果 



## loadrunner组件

## loadrunner脚本编写

### 参数化

#### 什么是参数化

脚本在发送请求的时候会提交一些数据，多个虚拟用户运行脚本时如果都提交相同的数据，这不符合实际的运行情况，而且可能会引发冲突。为了真实的体现真实的运行环境，需要尽可能的模拟各种各样的输入，这时候就用到参数化了。

#### 参数化的过程

1. 确定需要参数化的常量值
2. 设置参数的属性及数据源

##### 具体的操作

选中要参数化的内容。

方法一，右键---【**Replace with a new parameter**】
在脚本中找到需要参数化的值，选择，右键选择使用参数替换->新建参数
​	![img](http://upload-images.jianshu.io/upload_images/2936641-0a22757bbc8dc929.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击peoperties
![img](http://upload-images.jianshu.io/upload_images/2936641-a579bb83a089d505.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​	
方法二，右键---【**Use Existing Parameter**】

可以先把所有需要用到的参数化变量一起创建，在工具栏中点击参数化按钮，弹窗选项框，点击new按钮，创建新的参数化变量即可。


![图片.png](http://upload-images.jianshu.io/upload_images/2936641-aa87dfd7af035de6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建好之后，在脚本中需要参数化的变量上右键，选择--Use Existing Parameter --选择对应的变量。

#### 数据源来源之mysql数据库

具体内容看过来-->>[Loadrunner 参数化使用mysql数据源](http://kylin10.com/2017/02/06/Loadrunner%20%E5%8F%82%E6%95%B0%E5%8C%96%E4%BD%BF%E7%94%A8mysql%E6%95%B0%E6%8D%AE%E6%BA%90%20/) 



## loadrunner场景设置

## 录制手机端脚本

使用loadrunner也可以录制手机端的脚本，[戳我](http://kylin10.com/2017/01/19/loadrunner%E5%88%A9%E7%94%A8%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F%E5%BD%95%E5%88%B6%E6%89%8B%E6%9C%BA%E8%84%9A%E6%9C%AC/)

## loadrunner使用遇到的问题

### 常见报错信息

### 返回信息乱码解决方案

在使用loadrunner进行测试的过程中会发现服务器返回的中文信息不能正常显示，解决方法看这里 [戳我](http://kylin10.com/2017/02/06/%E8%BF%94%E5%9B%9E%E4%BF%A1%E6%81%AF%E4%B9%B1%E7%A0%81%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)