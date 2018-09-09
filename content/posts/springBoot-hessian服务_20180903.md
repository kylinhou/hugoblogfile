---
title: spring boot写hessian服务
categories: ["springboot"]
tags: ["springboot", "java","hessian"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---
# spring boot写hessian服务

## 添加hessian maven引用

```java
<dependency>    
      <groupId>com.caucho</groupId>    
       <artifactId>hessian</artifactId>    
        <version>4.0.38</version>
</dependency>
```

## 新建服务提供接口

新建接口 BillingService，接口有一个方法：downloadCostForAppStore()

```java
public abstract interface BillingService
{
    public abstract Response downloadCostForAppStore(AppStoreBillingRequest paramAppStoreBillingRequest);
}
```

## 实现服务

BillingService接口的实现类 BillingServiceHessianService

```java
@Service("BillingServiceHessianService")
public class BillingServiceHessianService implements BillingService{
    @Value("${billingService.wait.time}")
    private int billingServiceWaitTime;
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Override
    public  Response downloadCostForAppStore(AppStoreBillingRequest paramAppStoreBillingRequest) {
        Response response = new Response();
        ResulteCodeEnum resulteCodeEnum = ResulteCodeEnum.getResulteCodeByCode(0);
        response.setResulteCode(resulteCodeEnum);
        response.setReuslt(true);
        logger.info("hessian==============="+billingServiceWaitTime);
        try {
//          TimeUnit.SECONDS.sleep(1);//秒
            TimeUnit.MILLISECONDS.sleep(billingServiceWaitTime);//毫秒
        } catch (java.lang.InterruptedException e) {
            logger.error("线程等待出现问题");
        }


        return response;
    }

}
```

## 发布hessian服务

在 application的入口程序中发布服务

```
package com.bbkmobile.iqoo.bspread;

import com.bbkmobile.iqoo.bspread.charging.hessian.BillingService;
import com.bbkmobile.iqoo.bspread.charging.hessian.impl.BillingServiceHessianService;
import com.bbkmobile.iqoo.bspread.sender.rpcapi.sdk.CPDAPPListSender;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.remoting.caucho.HessianServiceExporter;

@SpringBootApplication
public class Cpddubbo2Application {

	@Autowired
	private BillingService billingService;

	@Autowired
	private CPDAPPListSender cpdappListSender;


	public static void main(String[] args) {
		SpringApplication.run(Cpddubbo2Application.class, args);
	}

	// 发布hessian服务
	@Bean(name = "/accountPay/downLoadCost.action")
	public HessianServiceExporter accountService() {
		HessianServiceExporter exporter = new HessianServiceExporter();
		exporter.setService(billingService);
		exporter.setServiceInterface(BillingService.class);
		return exporter;
	}

	// 发布服务
	@Bean(name = "/bspreadSpenderWeb/rpc/searchlist")
	public HessianServiceExporter cpdService() {
		HessianServiceExporter exporter = new HessianServiceExporter();
		exporter.setService(cpdappListSender);
		exporter.setServiceInterface(CPDAPPListSender.class);
		return exporter;
	}
}

```

以上即可完成hessian服务的编写。

## 测试hessian服务

通过一个ngrinder脚本测试下hessian服务是否正常。

下面是主要的代码片段。

```java
/// 1:定义请求的url	
public static  String url = "http://127.0.0.1:8080/accountPay/downLoadCost.action";
/// 2：根据url获取hessian服务
	public static  HessianProxyFactory factory = new HessianProxyFactory();
	public static  BillingService cpdRpcService = (BillingService)factory.create(BillingService.class,url);
/// 3：构造请求数据
	AppStoreBillingRequest appStoreBillingRequest = new AppStoreBillingRequest();
/// 4：发送请求
	Response resp = cpdRpcService.downloadCostForAppStore(appStoreBillingRequest);
```

