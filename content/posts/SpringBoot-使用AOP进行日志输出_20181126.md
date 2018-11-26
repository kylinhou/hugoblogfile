---
title: spring boot使用AOP进行日志输出
categories: ["springboot","java","AOP"]
tags: ["springboot", "redis"]
date: 2018-11-08
author: "kylin"
grammar_cjkRuby: true
---

# spring boot使用AOP进行日志输出

## Spring AOP介绍

Spring AOP专门介绍文章：

这里就不单独说AOP 了，只是做个简单的说明在spring boot中如何快速使用切面进行日志输出。

## 编写业务类

AOP的目的就是让程序员专心实现自己的业务逻辑，其他的公共事情交给其他应用对象来处理。先写一个简单的controller代表我们的业务。

```java
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AopController {

    @RequestMapping("/helloaop")
    public String sayHello() {
        return "hello AOP";
    }
}
```

一个很简单的controller，收到请求之后返回"hello AOP"。

返回"hello AOP"就是我们的业务，虽然我们想要在收到请求之前和之后打印日志，但是我们无需在业务类中进行任何操作，这些都交给AOP。

## 引入AOP依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 定义切面类

```java
package com.kylin.chapter4;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Arrays;


@Aspect
@Component
public class MyAspect {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 通过@Pointcut定义的切入点为com.kylin.chapter4包下的所有函数（对web层所有请求处理做切入点）
     */
    @Pointcut("execution(public * com.kylin.chapter4..*.*(..))")
    public void webLog(){}

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        // 接收到请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        // 记录下请求内容
        logger.info("URL : " + request.getRequestURL().toString());
        logger.info("HTTP_METHOD : " + request.getMethod());
        logger.info("IP : " + request.getRemoteAddr());
        logger.info("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
        logger.info("ARGS : " + Arrays.toString(joinPoint.getArgs()));

    }

    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable {
        // 处理完请求，返回内容
        logger.info("RESPONSE : " + ret);
    }

}
```

重新请求服务：/helloaop

看到控制台输出日志：

```xml
2018-11-08 16:26:31.397  INFO 13316 --- [nio-8080-exec-4] com.kylin.chapter4.MyAspect              : URL : http://127.0.0.1:8080/helloaop
2018-11-08 16:26:31.397  INFO 13316 --- [nio-8080-exec-4] com.kylin.chapter4.MyAspect              : HTTP_METHOD : GET
2018-11-08 16:26:31.397  INFO 13316 --- [nio-8080-exec-4] com.kylin.chapter4.MyAspect              : IP : 127.0.0.1
2018-11-08 16:26:31.397  INFO 13316 --- [nio-8080-exec-4] com.kylin.chapter4.MyAspect              : CLASS_METHOD : com.kylin.chapter4.AopController.sayHello
2018-11-08 16:26:31.397  INFO 13316 --- [nio-8080-exec-4] com.kylin.chapter4.MyAspect              : ARGS : []
2018-11-08 16:26:31.397  INFO 13316 --- [nio-8080-exec-4] com.kylin.chapter4.MyAspect              : RESPONSE : hello AOP
```

非常简单对不对？在spring boot中都无需使用`@EnableAspectJAutoProxy`来启用AOP。默认是开启的。

## 使用自定义注解

其实在上面已经实现了我们的需求，上面的方法定义表达式是通过一个规则表达式：

@Pointcut("execution(public * com.kylin.chapter4..*.*(..))")

这样在灵活性上差一点，下面介绍下通过使用自定义注解的方式实现同样的功能。

### 自定义注解

```java
package com.kylin.chapter4.annotation;
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AnalysisActuator {
    String note() default "";
}

```

#### 自定义注解说明

**一、注解的基础**

1.注解的定义：Java文件叫做Annotation，用@interface表示。

2.元注解：@interface上面按需要注解上一些东西，包括@Retention、@Target、@Document、@Inherited四种。

3.注解的保留策略：

　　@Retention(RetentionPolicy.SOURCE)   // 注解仅存在于源码中，在class字节码文件中不包含

　　@Retention(RetentionPolicy.CLASS)     // 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得

　　@Retention(RetentionPolicy.RUNTIME)  // 注解会在class字节码文件中存在，在运行时可以通过反射获取到

4.注解的作用目标：

　　@Target(ElementType.TYPE)                      // 接口、类、枚举、注解

　　@Target(ElementType.FIELD)                     // 字段、枚举的常量

　　@Target(ElementType.METHOD)                 // 方法

　　@Target(ElementType.PARAMETER)            // 方法参数

　　@Target(ElementType.CONSTRUCTOR)       // 构造函数

　　@Target(ElementType.LOCAL_VARIABLE)   // 局部变量

　　@Target(ElementType.ANNOTATION_TYPE) // 注解

　　@Target(ElementType.PACKAGE)               // 包

5.注解包含在javadoc中：

　　@Documented

6.注解可以被继承：

　　@Inherited

7.注解解析器：用来解析自定义注解。

### 在切面中使用注解定义切点

```java
    @Pointcut("@annotation(analysisActuator)")
    public void webLogAnnotation(AnalysisActuator analysisActuator){}


    @Before("webLogAnnotation(analysisActuator)")
    public void doBefore(JoinPoint joinPoint,AnalysisActuator analysisActuator) throws Throwable {
        // 接收到请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        // 记录下请求内容
        logger.info(analysisActuator.note());
        logger.info("URL : " + request.getRequestURL().toString());
        logger.info("HTTP_METHOD : " + request.getMethod());
        logger.info("IP : " + request.getRemoteAddr());
        logger.info("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
        logger.info("ARGS : " + Arrays.toString(joinPoint.getArgs()));
    }
```

### 业务代码中使用注解

```java
    @AnalysisActuator(note = "使用自定义注解")
    @RequestMapping("/helloaop")
    public String sayHello() {
        return "hello AOP";
    }
```

再次请求，看到输出日志：

```sh
2018-11-08 17:24:05.096  INFO 15088 --- [nio-8080-exec-3] com.kylin.chapter4.MyAspect              : 使用自定义注解
2018-11-08 17:24:05.096  INFO 15088 --- [nio-8080-exec-3] com.kylin.chapter4.MyAspect              : URL : http://127.0.0.1:8080/helloaop
2018-11-08 17:24:05.096  INFO 15088 --- [nio-8080-exec-3] com.kylin.chapter4.MyAspect              : HTTP_METHOD : GET
2018-11-08 17:24:05.096  INFO 15088 --- [nio-8080-exec-3] com.kylin.chapter4.MyAspect              : IP : 127.0.0.1
2018-11-08 17:24:05.096  INFO 15088 --- [nio-8080-exec-3] com.kylin.chapter4.MyAspect              : CLASS_METHOD : com.kylin.chapter4.AopController.sayHello
2018-11-08 17:24:05.096  INFO 15088 --- [nio-8080-exec-3] com.kylin.chapter4.MyAspect              : ARGS : []
2018-11-08 17:24:05.096  INFO 15088 --- [nio-8080-exec-3] com.kylin.chapter4.MyAspect              : RESPONSE : hello AOP
```

## AOP使用的总结

在spring boot中使用AOP的基本步骤：

1. 引入AOP依赖

2. 定义切面类

   @Aspect
   @Component

3. 切面类中定义切点

4. 在切面类中编写通知方法

在定义切点的时候可以使用自定义的注解。