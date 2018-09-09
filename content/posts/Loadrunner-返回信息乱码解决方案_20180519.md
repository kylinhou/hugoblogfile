---
title: loadrunner返回信息乱码解决方案
categories: ["性能测试"]
tags: ["性能测试", "loadrunner", "乱码"]
date: 2016-09-13
author: "kylin"
grammar_cjkRuby: true
---

# loadrunner返回信息乱码解决方案

## 问题描述

loadrunner是一个很强大的工具，但是对于中文的支持并不友好，从服务器返回的信息如果包含中文，显示将会是乱码。

本教程可以轻松查看乱码信息，使信息更直观。

<!--more-->

## 了解乱码

## 乱码带来的问题

1. 看不懂

2. 如果是关键信息，无法根据这个信息进行判断事务成功与否

   ```java
    	if( lr.eval_string("<result>").equals("true") ){
    		lr.end_transaction("openAT",lr.PASS);		
    	    }
    	else {
   	    lr.end_transaction("openAT",lr.FAIL);
   	    lr.error_message( lr.eval_string("<result2>") );
    	} 
   ```

   一般我们都是像上面一样去判断一个返回的值是不是跟我们预期的一致，但是如果是判断乱码的问题，就不能直接这样判断了，不管是跟正常的中文判断还是乱码之后的字符判断，都无法正常判断。

## 解决乱码

### c语言脚本

#### **lr_convert_string_encoding** 函数介绍

loadrunner中本身有一个函数 *lr_convert_string_encoding* ，可以用来查看乱码信息。

> **lr_convert_string_encoding** converts a string encoding between the following encodings: System locale, Unicode, and UTF-8. The function saves the result string, including its terminating NULL, in the parameter *paramName*. 

这是官方的解释，意思就是**lr_convert_string_encoding**() 函数可以将字符串在系统语言环境、Unicode和UTF-8

直接进行编码转换。

#### **lr_convert_string_encoding** 函数使用

```c
int lr_convert_string_encoding( const char sourceString*, const char fromEncoding, const char **toEncoding, const char *paramName); 
```

举个例子：

假设乱码的字符串为：

> 鎵嬫満鑾峰彇app鐢ㄦ埛璇勮淇℃伅鏃跺嚭閿\x99

鬼知道这是什么！

那么我们就用lr_convert_string_encoding()函数来处理一下，

``` c
lr_convert_string_encoding("鎵嬫満鑾峰彇app鐢ㄦ埛璇勮淇℃伅鏃跺嚭閿\x99",LR_ENC_UTF8,LR_ENC_SYSTEM_LOCALE,"str");
```

现在已经将字符串从LR_ENC_UTF8格式转换到LR_ENC_SYSTEM_LOCALE编码，并且将新的字符串保存到了str变量中，我们重新输出一下str变量看一下：

> str = 手机获取app用户评论信息时出错\x00

这样就OK了

### java脚本

在测试过程中，java脚本还是用的很多的，但是发现loadrunner没有提供可以直接在java脚本中进行字符串编码转换的函数。

嗯，肯定是因为java已经有自己的解决方案了，然后想起前端开发过程中经常解决的乱码问题，道理都是一样的，开始干。

##### 方法一：

我是这样想的，就把这个输出信息的控制台当作一个浏览器，为什么显示乱码了呢？肯定是这个浏览器在输出信息的时候额外做了一些手脚，把服务器返回的数据给转换了一下编码方式，导致信息不能正常显示。

我想到了java中字符串的getBytes()方法：

String的getBytes()方法是得到一个字串的字节数组，这是众所周知的。但特别要注意的是，本方法将返回该操作系统默认的编码格式的字节数组。

然后我实验了一下：

```java
byte[] br = lr.eval_string("<result>").getBytes();
String brStr2 = new String(br,"utf-8");
```

将想要查看的内容保存到result参数中，然后对其重新编码并保存到brStr2变量中，输出brStr2，发现确实解决了乱码的问题。

但是有一次在用这个方法的时候，发现使用web.reg_save_param()保存乱码的信息到result的时候什么都没有保存，我猜测是因为乱码中的一些特殊字符导致左右边境判断失败，所以无法正常的保存信息。所以就想另外一个方法。

##### 方法二：

既然我们都是使用web.reg_save_param()函数把相应的信息保存，这个函数本身有没有什么方法解决乱码问题呢？

去查询手册，发现了一个参数：

> - **Convert**:
>
>   > The possible values are:
>   > HTML_TO_URL:  convert HTML–encoded data to a 
>   > URL–encoded data format
>   > HTML_TO_TEXT: convert HTML–encoded data to plain text 
>   > format
>   > This attribute is optional.

当我看到   *HTML_TO_TEXT: convert HTML–encoded data to plain text  format* 也就是将HTML编码的数据转换为纯文本格式 的时候，感觉有希望！

然后添加上这个参数，并设置值为HTML_TO_TEXT，然后对结果重新进行编码，结果无论采用哪种编码方式，看到的仍然是乱码的。

然后试了一下另外一个值：HTML_TO_URL，发现是可以的！！！而且非常有效，建议以后可以采取这种方式，非常方便。

首先，引入相应的包，因为要用到URLdecode和encode的一些方法，直接在头部添加一行代码就OK：

```java
import java.net.URLDecoder;
```

然后在平时使用的web.reg_save_param()函数中多加一行代码就OK

```java
 web.reg_save_param ("result",
                     new String []{
                       "NOTFOUND=ERROR",
                       "LB=\"prompt1\":\"",
                       "RB=\"" ,
                       "Convert=HTML_TO_URL",
                       "LAST"} ); 
```

因为多了这么一行代码，那么穿过来的信息就被转换成了URL-encode之后的编码格式，我们看到的信息将是下图中的样子，直接看也是看不懂的，我们只要decode一下就可以了。

```java
String str = URLDecoder.decode(lr.eval_string("<result>"),"utf-8");
System.out.println("结果是："+str );
```

![](http://upload-images.jianshu.io/upload_images/2936641-f3db00d1bf1478af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

经过实验发现这种方法非常有效，而且直观，也可以针对中文进行结果的判断。



