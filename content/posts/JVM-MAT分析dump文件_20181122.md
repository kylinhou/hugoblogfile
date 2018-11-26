---
title: MAT分析dump文件
categories: ["JVM"]
tags: ["性能测试", "JVM","MAT"]
date: 2018-11-22
author: "kylin"
grammar_cjkRuby: true
---

# MAT分析dump文件

## Heap Dump

Heap Dump：堆转储文件，生成 Java™ 堆上所有活动对象（即，正在运行的 Java 应用程序正在使用的对象）的转储

> 转储格式有两种，文本或经典堆转储格式和可移植堆转储 (PHD) 格式。文本或经典格式可由人类阅读，但是 PHD 格式是压缩的，无法由人类阅读。这两种格式都包含堆中所有对象实例的列表，包括每个对象的地址、类型或类名、大小以及对其他对象的引用。这些堆转储还包含有关生成该转储的 JVM 版本的信息。它们不包含除类名和引用的值（地址）外的任何对象内容或数据。

## MAT介绍及安装

在使用jmap命令获取到jvm的dump信息之后，我们要进行分析，MAT是其中一个比较好用的工具。

> jmap -dump:format=b,file=/my.dump pid

IBM还有一个分析工具，但是使用起来不是很友好，推荐使用MAT。

### Eclipse Memory Analyzer(MAT)

[Eclipse Memory Analyzer(MAT)](http://www.eclipse.org/mat/)是Eclipse提供的一款用于Heap Dump文件的工具，它支持两种安装方式，一是Eclipse插件的方式，另外一个就是独立运行的方式，建议使用独立运行的方式。

## MAT使用

下载之后解压可以直接运行，打开软件之后：

打开 File - Open Heap Dump... 菜单，然后选择生成的Heap DUmp文件，选择 "Leak Suspects Report"，然后点击 "Finish" 按钮。

打开Dump文件可能要很久，文件越大时间越久。

### Overview视图

这是打开之后的主页面：Overview视图

![](https://i.loli.net/2018/11/22/5bf61cd6784a7.png)

Overview视图，即概要界面，显示了概要的信息，并展示了MAT常用的一些功能。

- Details 显示了一些统计信息，包括整个堆内存的大小、类（Class）的数量、对象（Object）的数量、类加载器（Class Loader)的数量。

  例如：Size: 870.9 MB， Classes: 16.6k ，Objects: 5m， Class Loader: 471 

  Unreachable Objects Histogram 连接进去可以查看

- Biggest Objects by Retained Size 使用饼图的方式直观地显示了在JVM堆内存中最大的几个对象，当光标移到饼图上的时候会在左边Inspector和Attributes窗口中显示详细的信息。

- Actions 这里显示了几种常用到的操作，算是功能的快捷方式，包括 Histogram、Dominator Tree、Top Consumers、Duplicate Classes，具体的含义和用法见下面；

  - Histogram：直方图，可以查看每个类的实例（即对象）的数量和大小
  - Dominator Tree：支配树，列出Heap Dump中处于活跃状态中的最大的几个对象，默认按 retained size进行排序，因此很容易找到占用内存最多的对象。

- Reports 列出了常用的报告信息，包括 Leak Suspects和Top Components，具体的含义和内容见下；

- Step By Step 以向导的方式引导使用功能。

英语不好，来个翻译：

* Histogram：英[ˈhɪstəgræm] 柱状图
* Dominator ：英['dɒmɪneɪtə] 支配者，支配力
* Duplicate ：英[ˈdju:plɪkeɪt] 重复; 复制; 复印

### Histogram直方图

先说两个概念：

*  Shallow Heap ：对象自身占用的内存大小，不包括它引用的对象

* Retained Heap：Retained Size=当前对象大小+当前对象可直接或间接引用到的对象的大小总和。

  (间接引用的含义：A->B->C, C就是间接引用) 
  换句话说，Retained Size就是当前对象被GC后，从Heap上总共能释放掉的内存。

Histogram直方图中显示的数据的默认单位是Bytes，可以在Window - Preferences 菜单中设置单位。

通过Retained Heap排序，可以找到占用内存最多的几个类。

![](https://i.loli.net/2018/11/22/5bf64ed015cad.png)

并且可以对比两个dump文件来进行分析，如果存在内存溢出，时间久了溢出类的实例数量或者内存占比会越来越多，排名也越来越靠前。所以在分析的时候可以在不同的时间节点dump两份文件。

### Dominator Tree（支配树）视图

在工具栏打开Dominator Tree（支配树）视图，在此视图中列出了每个对象（Object Instance）与其引用关系的树状结构，同时包含了占用内存的大小和百分比。

![](https://i.loli.net/2018/11/22/5bf6500877c6a.png)

通过Dominator Tree视图可以很容易的找出占用内存最多的几个对象（根据Retained Heap或Percentage排序），和Histogram类似，可以通过不同的方式进行分组显示。

### 定位问题代码

#### 方法一：

Histogram视图和Dominator Tree视图的角度不同，前者是基于类的角度，后者是基于对象实例的角度，并且可以更方便的看出其引用关系。

首先，在两个视图中找出疑似溢出的对象或者类（可以通过Retained Heap排序，并且可以在Class Name中输入正则表达式的关键词只显示指定的类名），然后右键选择Path To GC Roots（Histogram中没有此项）或Merge Shortest Paths to GC Roots，然后选择 *exclude all phantom/weak/soft etc. reference*：

![1542869283385](C:/Users/11041385/AppData/Roaming/Typora/typora-user-images/1542869283385.png)

GC Roots意为GC根节点，其含义见上面的 [GC Roots和Reference Chain](https://www.javatang.com/archives/2017/11/08/11582145.html#GC_RootsReference_Chain) 部分，后面的 *exclude all phantom/weak/soft etc. reference* 意思是排除[虚引用、弱引用和软引用](https://www.javatang.com/archives/2017/11/08/11582145.html#Reference)，即只剩下[强引用](https://www.javatang.com/archives/2017/11/08/11582145.html#Reference)，因为除了强引用之外，其他的引用都可以被JVM GC掉，如果一个对象始终无法被GC，就说明有强引用存在，从而导致在GC的过程中一直得不到回收，最终就内存溢出了。

通过结果就可以很方便的定位到具体的代码，然后分析是什么原因无法释放该对象，比如被缓存了或者没有使用单例模式等等。

![](https://i.loli.net/2018/11/22/5bf65176c5772.png)

### 方法二

MAT工具自动会帮助我们进行分析，在OverView视图的Reports 部分，有一个Leak Suspects连接，直接进去

![](https://i.loli.net/2018/11/22/5bf653632103f.png)

进去之后就可以看到工具帮我们分析到的可疑点：

![](https://i.loli.net/2018/11/22/5bf653d39ec0c.png)

在详情里面看到具体的代码

![](https://i.loli.net/2018/11/22/5bf654334b413.png)

两种方式定位到的代码是相同的，都可以找到问题点。

## 参考文章

* https://www.javatang.com/archives/2017/11/08/11582145.html
* https://www.ibm.com/support/knowledgecenter/zh/SSYKE2_8.0.0/com.ibm.java.vm.80.doc/docs/heapdump.html
* https://www.ibm.com/developerworks/cn/opensource/os-cn-ecl-ma/





