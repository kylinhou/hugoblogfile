---
title: nGrinder使用手册
categories: ["性能测试"]
tags: ["性能测试", "ngrinder"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---

# nGrinder使用手册


## 一：它是谁，它从哪里来，它可以做什么

nGrinder为什么叫nGrinder，因为是基于Grinder开源项目开发实现。

是NAVER（韩国最大互联网公司NHN旗下搜索引擎网站）开源的性能测试工具。

nGrinder是一款非常易用，有友好简洁的用户界面和controller-agent分布式结构的强大的压力测试工具。

> 它是由一个controller和连接它的多个agent组成，用户可以通过web界面管理和控制测试，以及查看测试报告，controller会把测试分发到一个或多个agent去执行。用户可以设置使用多个进程和线程来并发的执行该脚本，而且在同一线程中，来重复不断的执行测试脚本，来模拟很多并发用户。
>
> nGrinder的测试是基于一个python的测试脚本，用户按照一定规则编写测试脚本以后，controller会将脚本以及需要的其他文件分发到agent，用Jython执行。并在执行过程中收集运行情况、响应时间、测试目标服务器的运行情况等。并保存这些数据生成运行报告，以供以后查看。

![ngrinder架构](http://img.blog.csdn.net/20160225193628121)

## 二：nGrinder的组件介绍

nGrinder可以简单分为3部分：

* Controller：为性能测试提供web interface、协同测试进程、收集和显示测试数据、管理测试脚本
* Agent：运行进程和线程，压测目标服务
* Monitor：监控目标系统性能(cpu/memory)， 可以自定义收集的数据(比如 jvm数据)

## 三：nGrinder的安装

nGrinder的安装与部署部分简单介绍一下，不做详细说明。

nGrinder是一个web应用(Controller)和Java应用(Agent, Monitor)的组合。安装 nGrinder的Controller和Agent，需要安装JDK 1.6或更高的版本。.

nGrinder需要用到很多端口。如果有些端口被防火墙阻挡，请联系服务器管理开放下面这些端口。

- Agent : Any ==> Controller : 16001
- Agent : Any ==> Controller : 12000 ~ 12000+(允许并发测试的数量)
- Controller : Any ==> Monitor : 13243

### 下载

可以从这里下载 ：https://sourceforge.net/projects/ngrinder/files/

最新版本 [ngrinder-3.3](https://sourceforge.net/projects/ngrinder/files/ngrinder-3.3/)

下载 [ngrinder-controller-3.3.war](https://sourceforge.net/projects/ngrinder/files/ngrinder-3.3/ngrinder-controller-3.3.war/download)

有了war包，nGrinder就可以跟其他的java web工程一样部署到tomcat里面，然后启动。

除此之外，nGrinder还可以直接用命令行启动: 

> java -jar ngrinder-controller-X.X.war

### 部署controller

**以在Tomcat中运行为例简单说明部署步骤**

1. 将war包文件放到tomcat的webapps文件夹中，${TOMCAT_HOME}/webapps 。如果你不想通过路径ngrinder-controller访问nGrinder，可以修改war包文件的名称为war.
2. 然后打开sh或者catalina.bat将下面这行命令添加到文件头部。

```
JAVA_OPTS="-Xms600m -Xmx1024m -XX:MaxPermSize=200m"    # for catalina.sh
set JAVA_OPTS=-Xms600m -Xmx1024m -XX:MaxPermSize=200m   # for catalina.bat
```

1. 然后运行${TOMCAT_HOME}/startup.sh或者bat
2. 打开浏览器访问http://ip:port/ngrinder-controller-X.X或者http://ip:port如果你修改了war包文件的名称为war

体验入口：http://172.25.1.16:6703/ngrinder/login  用户名:guest 密码：vivo123456

### 下载agent

![下载agent](http://img.ispenn.com/wp-content/uploads/2015/08/ngrinder-image-3.png)

进入controller可以下载agent，agent下载下来是一个tar包，上传tar包到准备好的压力机上，解压包并且运行其中的sh或者run_agent.bat即可。

> [ngrinder@localhost ~]$ cd ngrinder-agent
> [ngrinder@localhost ngrinder-agent]$ ll
> total 40
> -rw-r--r-- 1 ngrinder root     1733 Apr 19 20:02 __agent.conf
> drwxrwxr-x 2 ngrinder ngrinder 4096 Apr 17 22:02 dubbo
> drwxr-xr-x 2 ngrinder root     4096 Apr 18 10:46 lib
> -rwxr-xr-x 1 ngrinder root      367 Jun 17  2017 run_agent.bat
> -rwxr-xr-x 1 ngrinder root       83 Jun 17  2017 run_agent_bg.sh
> -rwxr-xr-x 1 ngrinder root      237 Jun 17  2017 run_agent_internal.bat
> -rwxr-xr-x 1 ngrinder root       99 Jun 17  2017 run_agent_internal.sh
> -rwxr-xr-x 1 ngrinder root      312 Jun 17  2017 run_agent.sh
> -rwxr-xr-x 1 ngrinder root      135 Jun 17  2017 stop_agent.bat
> -rwxr-xr-x 1 ngrinder root      136 Jun 17  2017 stop_agent.sh

其中 __agent.conf 是agent的配置文件，主要的配置项说明：

1. agent.controller_host=172.25.1.16   controller 的ip

2. agent.region=NONE

3. agent.host_id=172.25.34.11 agent的ip

   说明：

   * agent.region的默认配置为NONE ,标明改负载机可以被所有的账户使用，如果想要设置该agent只能被指定用户使用，配置格式为：NONE_owned_name：如 agent.region=NONE_owned_xlhou2
   * agent.host_id 可以不做配置，不做配置的话默认线上服务器的hostname。


注：修改agent的配置之后重新启动需要添加 -o 参数。o: overwrite

### monitor安装与配置

1. 下载monitor。 
   ![](http://img.ispenn.com/wp-content/uploads/2015/08/ngrinder-image-5.png)


1. 解压monitor的包并运行下面的命令：

```
run_monitor_bg.sh  # for linux / mac
run_monitor.bat # for windows
```

2. 停止monitor，运行下面的命令：

```
stop_monitor.sh # for linux / mac
stop_monitor.bat –o # for windows
```

## 四：使用nGrinder快速进行简单测试

使用nGrinder进行性能测试的完整步骤：

1. 编写脚本
2. 设计场景：配置虚拟用户数，周期，步长控制，资源监控；
3. 运行结束报告自动生成

对于简单的测试可以让nGrinder会自动生成测试脚本

![1524897814690](/1524897814690.png)

如果需要对一个接口进行简单的压测，接口url是固定的，请求参数无需改动，可以直接快速压测，无需手动写脚本。

进入nGrinder首页，在quick start右边的填写框里面写上待测试的url，例如

http://172.25.34.6:3001/qqServerSwitch.do

然后点击 start test

![1524898143010](/1524898143010.png)

填写相关信息，然后点击save and test就可以进行开始测试了。脚本都不需要写，是不是很方便？

当然对于大部分脚本需要自己手动编写，设置参数化、设置断言。具体的脚本编写后面介绍。

## 五：nGrinder脚本编写

nGrinder的脚本可以使用groovy语言，可以使用python语言。

下面简单介绍下脚本的基本内容，具体可以下载脚本参考。

脚本下载地址：

如果使用groovy语言建议选择groovy maven工程，也就是在下拉框中选择 groovy maven project

看下工程的目录结构

![1524905025100](/1524905025100.png)

工程结构跟普通的java maven工程是一样的，默认的工程主要的测试代码只有一个TestRunner文件

主要的代码如下，非常的简单：

```java
	@Test
	public void test(){
		HTTPResponse result = request.GET("http://172.25.34.6:3001/qqServerSwitch.do", params)

		if (result.statusCode == 301 || result.statusCode == 302) {
			grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", result.statusCode); 
		} else {
			assertThat(result.statusCode, is(200));
		}
	}
```

自动生成的脚本是检查输入的链接是不是返回200，可以根据自己的需要修改脚本。

### 一：传参

#### 基本参数

ngrinder发送http请求时，传参都是通过一个数组 params

```java
// 1:类变量声明
public static NVPair[] params = []

// 2:请求之前设置参数
// 设置请求参数
List<NVPair> paramList = new ArrayList<NVPair>()
		paramList.add(new NVPair("id", id))
		paramList.add(new NVPair("type", type))
            
 params = paramList.toArray()
//3：发请求
    HTTPResponse result = request.GET("http://172.25.34.6:2820/api19.do", params)
```

#### post 请求发送带json格式的内容

```java
// json字符串
        def json = '{"tenant_code":"XXX","user_name":"XX","password":"X","skip_duplicate_entries":true,"type":"0"}';
// 发起请求
HTTPResponse result = request.POST("http://192.XX/XX/login", json.getBytes(), headers());
```

#### 添加请求头

```java
////根据需求添加请求头信息
List<NVPair> headerList = new ArrayList<NVPair>()
headerList.add(new NVPair("test_header", "test_header_value"))
headers = headerList.toArray()
////发请求
HTTPResponse result = request.GET("http://172.25.34.6:2820/api19.do", params, headers() )
```

例如设置 接口请求 的 Content-Type 类型

> ```
> headerList.add(new NVPair("Content-Type", "application/json"))
> ```

### 二：上传文件

待补充

### 三：参数化

####  文件参数化之唯一取值

```java
String[] getParam(String [] ParamCsv,int processNumber,int threadNumber,int agentNumber,int curCount) {

        int totalThreadCount
        int totalProcessCount
        if(grinder.getProperties() == null ) {
            totalProcessCount = 1;//grinder.getProperties().getInt("grinder.processes", 1); //总共并发执行的进程数，数量
            totalThreadCount = 100;//grinder.getProperties().getInt("grinder.threads", 1);////一个进程下总共的线程数，数量
        } else {
            totalThreadCount = grinder.getProperties().getInt("grinder.threads", 1);//grinder.getProperties().getInt("grinder.threads", 1);////一个进程下总共的线程数，数量
            totalProcessCount = grinder.getProperties().getInt("grinder.processes", 1);//grinder.getProperties().getInt("grinder.processes", 1); //总共并发执行的进程数，数量
        }

        String[] CsvColumn ;
        if( ParamCsv.length == 0 ) {
            ////解决当用不到参数化文件时，ParamCsv.length=0带来的问题
            CsvColumn = null;
        } else {
             int curTestNum = (curCount * totalProcessCount * totalThreadCount) + (processNumber * totalThreadCount) + threadNumber;/////总执行过程中的一个编号，所有线程的所有循环统一编号下的一个编号
            int agentCsvLen = ParamCsv.length/agentNumber////ParamCsv.length的值是大于等于参数个数的agentNumber的最小倍数。用来确保参数个数可以被代理机器整出的
            int index = (agentNumber-1)*agentCsvLen + curTestNum%agentCsvLen;
            String tempStr = ParamCsv[index];//取csv文件的哪一行
            CsvColumn = tempStr.split(",");
        }
        return CsvColumn
    }
```

#### 文件参数化之随机取值

```java
    /**
     * 随机从文件中获取一行，进行参数化
     * @param ParamCsv
     * @return
     */
    public String[] getRandomFromFile(String [] ParamCsv) {
        RandomNumber randomNumber = new RandomNumber();
        int index = randomNumber.getRandomNumber(0,ParamCsv.size());
        String parms = ParamCsv[index];
        String[] CsvColumn = parms.split(",");
        return CsvColumn;
    }
```

#### 自然数之随机取值

获取随机数

getRandomNumber(int 12,int 500)：返回12到500之间的随机数

```java
    int getRandomNumber(int min,int max) {
        Random random = new Random();
        int s = random.nextInt(max)%(max-min+1) + min;
        return s
    }
```

如果需要返回的是一定长度的编码，如 00015

只需调用 autoGenericCode(int 15, int 5)

```java
    /**
     * 不够位数的在前面补0，保留size的长度位数字
     * @param code
     * @return
     */
    public String autoGenericCode(int code, int size) {
        String result = "";
        ////num 指定长度
        result = String.format("%0" + size + "d", code);
        return result;
    }
```

#### 自然数之唯一取值

getUniqueCode返回的是指定格式的编码

```
openid = "1234567"+randomNumber.getUniqueCode(8,processNumber,threadNumber,agentNumber,curCount)
```

那么openid就是 123456700000000、123456700000001、123456700000002递增


```java
public String  getUniqueCode(int codeSize,int processNumber,int threadNumber,int agentNumber,int curCount){
        int totalThreadCount
        int totalProcessCount
        String num;
        if(grinder.getProperties() == null ) {
            // 这时候是在本地调试代码
            totalProcessCount = 1;
            totalThreadCount = 100;
        } else {
            //压测时
            ////一个进程下总共的线程数，数量
            totalThreadCount = grinder.getProperties().getInt("grinder.threads", 1);
            //总共并发执行的进程数，数量
            totalProcessCount = grinder.getProperties().getInt("grinder.processes", 1); 
        }
        int curTestNum = (curCount * totalProcessCount * totalThreadCount) + (processNumber * totalThreadCount) + threadNumber;/////总执行过程中的一个编号，所有线程的所有循环统一编号下的一个编号
        num = autoGenericCode(curTestNum,codeSize)
        return num;
    }
```

如果只是需要简单的数字，对长度无要求，可以用 getUniqueNum

```java
public int  getUniqueNum(int processNumber,int threadNumber,int agentNumber,int curCount){
        int totalThreadCount
        int totalProcessCount

        if(grinder.getProperties() == null ) {
            // 这时候是在本地调试代码
            totalProcessCount = 1;
            totalThreadCount = 100;
        } else {
             //压测时
            ////一个进程下总共的线程数，数量
            totalThreadCount = grinder.getProperties().getInt("grinder.threads", 1);
             //总共并发执行的进程数，数量
            totalProcessCount = grinder.getProperties().getInt("grinder.processes", 1);

        }
        int curTestNum = (curCount * totalProcessCount * totalThreadCount) + (processNumber * totalThreadCount) + threadNumber;/////总执行过程中的一个编号，所有线程的所有循环统一编号下的一个编号

        return curTestNum;
    }
```

### 四：断言

可以简单判断标志性语句是否存在，也可以强校验出现次数

```java
String target = "\"code\":0";
int times = judgeResult.findOccurrence(target,respData);
if ( times == 1) {
    Assert.assertTrue(true)
} else {
	grinder.logger.error("data:"+respData);
	Assert.fail("request it's failed !!")
}
```

## 六：场景设计

一张图就够了

![1526903315639](/1526903315639.png)

 ## 七：agent设置

### agent的配置文件

上传agent jar包到服务器上解压 ngrinder-agent-3.4.1-172.25.34.10.tar

文件：

>  __agent.conf
>  dubbo
>  lib
>  run_agent.bat
>  run_agent_bg.sh
>  run_agent_internal.bat
>  run_agent_internal.sh
>  run_agent.sh
>  stop_agent.bat
>  stop_agent.sh

配置文件主要配置说明：

vim  __agent.conf ：

```shell
common.start_mode=agent
agent.controller_host=172.25.1.16 #配置controller的ip 
agent.controller_port=16001 #与controller通信的端口号，默认
agent.region=NONE_owned_xlhou  #使用权限配置
agent.host_id=172.25.34.11 #在controller中显示的本机名称
```

如果配置 agent.region=NONE ，所有用户都可以使用该负载机，建议添加权限控制。

###　配置ulimit 运行更多线程

如果agent运行在Linux下，你可能需要配置ulimit让其运行更多的线程。请检查下面的配置。

>ulimit -a
>core file size          (blocks, -c) 0
>data seg size          (kbytes, -d) unlimited
>scheduling priority            (-e) 0
>file size              (blocks, -f) unlimited
>pending signals                (-i) 30676
>max locked memory      (kbytes, -l) 64
>max memory size        (kbytes, -m) unlimited
>open files                      (-n) 16000
>pipe size            (512 bytes, -p) 8
>POSIX message queues    (bytes, -q) 819200
>real-time priority              (-r) 0
>stack size              (kbytes, -s) 10240
>cpu time              (seconds, -t) unlimited
>max user processes              (-u) 32768
>virtual memory          (kbytes, -v) unlimited
>file locks                      (-x) unlimited

注意查看 max user processes   的值，如果太小是不行的

修改max user processes

1. 使用root用户修改配置文件：/etc/security/limits.conf 

   增加如下内容

   \* soft nproc 32768

   \* hard nproc 32768

   \* soft nofile 16000

   \* hard nofile 16000

   其中nofile对应open_files

   nproc对应max_user_processes

2. 修改/etc/security/limits.d/90-nproc.conf 

   将 * soft nproc 1024  修改为：

   \* soft nproc 32768

## 八：常见问题及解决方案

#### 1：脚本已经更新了，为什么从之前压测的场景链接到脚本查看，依然是旧代码？

答：ngrinder会保存每一次压测时的脚本信息，如果脚本更新了，从已经压测结束的场景里面链接到的脚本无法查看最新代码，需要 复制场景为新的待压测场景，下次压测将使用新的代码。

#### 2：为什么不能同时开始两个压测？

答：ngrinder有限制，同一个用户在同时只能有一个场景压测。如果需要同时跑两个场景，可以使用其他账户一起压测。

#### 3：为什么分发文件特别慢

答：ngrinder有安全分发机制，当检测到需要分发的文件达到一定大小，将会开启安全分发模式，每隔5s分发一个文件，所以在超过安全限制之后，文件越多分发越慢，分发慢的原因是分发每个文件之前需要等待一段时间，而不是分发文件本身慢。

ngrinder默认安全大小 1000000 字节，目前，我已经修改为15M，如有需要可以继续修改。修改方法：

配置文件中修改如下配置项，然后重启服务：

>  controller.safe_dist_threshold=1000000

#### 4：异常测试删除停止方法

异常测试无法正常删除，场景的选择框无法选择

目前没有比较好的方式，可以采用编辑网页html代码的方式，删除disabled标签，然后选中删除。

## 九：已完善功能

1. 脚本选择框加大
2. 常用参数化方式

## 十：待做事项

1. 异常测试可以随意删除
2. 添加JVM监控

