---
title: mongo索引优化
categories: ["mongo"]
tags: ["编程", "mongo"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---

# mongo索引优化

## 问题描述

在测试某个接口的时候，性能非常差，线程状态都是与mongo有关系。具体的线程信息：

>名称: catalina-exec-109
>状态: RUNNABLE
>总阻止数: 28, 总等待数: 229
>
>堆栈跟踪:
>java.net.SocketInputStream.socketRead0(Native Method)
>java.net.SocketInputStream.read(SocketInputStream.java:152)
>java.net.SocketInputStream.read(SocketInputStream.java:122)
>java.io.BufferedInputStream.fill(BufferedInputStream.java:235)
>java.io.BufferedInputStream.read1(BufferedInputStream.java:275)
>java.io.BufferedInputStream.read(BufferedInputStream.java:334)
>   - 已锁定 java.io.BufferedInputStream@7ff42fbc
>       org.bson.io.Bits.readFully(Bits.java:73)
>         org.bson.io.Bits.readFully(Bits.java:50)
>         org.bson.io.Bits.readFully(Bits.java:37)
>         com.mongodb.Response.<init>(Response.java:42)
>         com.mongodb.DBCollectionImpl.receiveWriteCommandMessage(DBCollectionImpl.java:648)
>         com.mongodb.DBCollectionImpl.access$400(DBCollectionImpl.java:50)
>         com.mongodb.DBCollectionImpl$4.execute(DBCollectionImpl.java:580)
>         com.mongodb.DBCollectionImpl$4.execute(DBCollectionImpl.java:567)
>         com.mongodb.DBPort.doOperation(DBPort.java:187)
>   - 已锁定 com.mongodb.DBPort@786a2144
>       com.mongodb.DBTCPConnector.doOperation(DBTCPConnector.java:208)
>         com.mongodb.DBCollectionImpl.writeWithCommandProtocol(DBCollectionImpl.java:567)
>         com.mongodb.DBCollectionImpl.updateWithCommandProtocol(DBCollectionImpl.java:562)
>         com.mongodb.DBCollectionImpl.updateImpl(DBCollectionImpl.java:289)
>         com.mongodb.DBCollection.update(DBCollection.java:250)
>         com.mongodb.DBCollection.update(DBCollection.java:232)
>         com.mongodb.DBCollection.update(DBCollection.java:307)
>         org.springframework.data.mongodb.core.MongoTemplate$12.doInCollection(MongoTemplate.java:1157)
>         org.springframework.data.mongodb.core.MongoTemplate$12.doInCollection(MongoTemplate.java:1137)
>         org.springframework.data.mongodb.core.MongoTemplate.execute(MongoTemplate.java:463)
>         org.springframework.data.mongodb.core.MongoTemplate.doUpdate(MongoTemplate.java:1137)
>         org.springframework.data.mongodb.core.MongoTemplate.updateFirst(MongoTemplate.java:1119)
>         com.vivo.music.common.dal.dao.impl.MongoBaseDAOImpl.update(MongoBaseDAOImpl.java:75)
>         com.vivo.music.core.user.playlist.self.service.impl.PlaylistServiceImpl.delete(PlaylistServiceImpl.java:289)
>         com.vivo.music.web.controller.user.self.SelfPlaylistController.delete(SelfPlaylistController.java:297)
>         sun.reflect.GeneratedMethodAccessor383.invoke(Unknown Source)
>         sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
>         java.lang.reflect.Method.invoke(Method.java:606)
>         org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:220)
>         org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:134)
>         org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:116)
>         org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:827)
>         org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:738)
>         org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:85)
>         org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:963)
>         org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:897)
>         com.vivo.framework.spring.webmvc.VivoDispatcherServlet.doService(VivoDispatcherServlet.java:96)
>         org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970)
>         org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:872)
>         javax.servlet.http.HttpServlet.service(HttpServlet.java:650)
>         org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846)
>         javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
>         org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:303)
>         org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
>         org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
>         org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
>         org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
>         com.vivo.music.web.common.filter.CustomServletFilter.doFilter(CustomServletFilter.java:85)
>         org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
>         org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
>         com.vivo.internet.codec.CodecServletFilter.doFilter(CodecServletFilter.java:114)
>         org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
>         org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
>         org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:197)
>         org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
>         org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
>         org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
>         org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:218)
>         org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:122)
>         org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:505)
>         org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:169)
>         org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:103)
>         org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:116)
>         org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:442)
>         org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1082)
>         org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:623)
>         org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1756)
>         org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1715)
>   - 已锁定 org.apache.tomcat.util.net.NioChannel@76d17d0a
>       java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
>         java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
>         org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
>         java.lang.Thread.run(Thread.java:745)

### 解决方法

在进行mongo插入操作的时候性能是非常好的，不知道为什么在进行根据某个字段修改信息时，性能这么差，原因是查询的字段在mongo中没有添加索引。添加索引之后问题里面解决。

## MongoDB索引

### 1. 创建／重建索引

MongoDB全新创建索引使用`ensureIndex()`方法，对于已存在的索引可以使用`reIndex()`进行重建。

#### 1.1 创建索引`ensureIndex()`

MongoDB创建索引使用`ensureIndex()`方法。

**语法结构**

```
db.COLLECTION_NAME.ensureIndex(keys[,options])
```

- `keys`，要建立索引的参数列表。如：`{KEY:1}`，其中`key`表示字段名，`1`表示升序排序，也可使用使用数字`-1`降序。

- `options`，可选参数，表示建立索引的设置。可选值如下：

- - `background`，Boolean，在后台建立索引，以便建立索引时不阻止其他数据库活动。默认值 false。
  - `unique`，Boolean，创建唯一索引。默认值 false。
  - `name`，String，指定索引的名称。如果未指定，MongoDB会生成一个索引字段的名称和排序顺序串联。
  - `dropDups`，Boolean，创建唯一索引时，如果出现重复删除后续出现的相同索引，只保留第一个。
  - `sparse`，Boolean，对文档中不存在的字段数据不启用索引。默认值是 false。
  - `v`，index version，索引的版本号。
  - `weights`，document，索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。

如，为集合`sites`建立索引：

```
> db.sites.ensureIndex({name: 1, domain: -1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```

*注意：*`1.8`版本之前创建索引使用`createIndex()`，`1.8`版本之后已移除该方法

#### 1.2 重建索引`reIndex()`

```
db.COLLECTION_NAME.reIndex()
```

如，重建集合`sites`的所有索引：

```
> db.sites.reIndex()
{
  "nIndexesWas" : 2,
  "nIndexes" : 2,
  "indexes" : [
    {
	  "key" : {
		"_id" : 1
	  },
	  "name" : "_id_",
		"ns" : "newDB.sites"
	},
	{
	  "key" : {
		"name" : 1,
		"domain" : -1
	  },
	  "name" : "name_1_domain_-1",
	  "ns" : "newDB.sites"
	}
  ],
  "ok" : 1
}
```

### 2. 查看索引

MongoDB提供了查看索引信息的方法：`getIndexes()`方法可以用来查看集合的所有索引，`totalIndexSize()`查看集合索引的总大小，`db.system.indexes.find()`查看数据库中所有索引信息。

#### 2.1 查看集合中的索引`getIndexes()`

```
db.COLLECTION_NAME.getIndexes()
```

如，查看集合`sites`中的索引：

```
>db.sites.getIndexes()
[
  {
	"v" : 1,
	"key" : {
	  "_id" : 1
	},
	"name" : "_id_",
	"ns" : "newDB.sites"
  },
  {
	"v" : 1,
	"key" : {
	  "name" : 1,
	  "domain" : -1
	},
	"name" : "name_1_domain_-1",
	"ns" : "newDB.sites"
  }
]
```

#### 2.2 查看集合中的索引大小`totalIndexSize()`

```
db.COLLECTION_NAME.totalIndexSize()
```

如，查看集合`sites`索引大小：

```
> db.sites.totalIndexSize()
16352
```

#### 2.3 查看数据库中所有索引`db.system.indexes.find()`

```
db.system.indexes.find()
```

如，当前数据库的所有索引：

```
> db.system.indexes.find()
```

### 3. 删除索引

不在需要的索引，我们可以将其删除。删除索引时，可以删除集合中的某一索引，可以删除全部索引。

#### 3.1 删除指定的索引`dropIndex()`

```
db.COLLECTION_NAME.dropIndex("INDEX-NAME")
```

如，删除集合`sites`中名为"name_1_domain_-1"的索引：

```
> db.sites.dropIndex("name_1_domain_-1")
{ "nIndexesWas" : 2, "ok" : 1 }
```

#### 3.3 删除所有索引`dropIndexes()`

```
db.COLLECTION_NAME.dropIndexes()
```

如，删除集合`sites`中所有的索引：

```
> db.sites.dropIndexes()
{
  "nIndexesWas" : 1,
  "msg" : "non-_id indexes dropped for collection",
  "ok" : 1
}
```

## mongo查询诊断

 explian()能够提供大量与查询相关的信息。对于速度比较慢的查询来说，这是最重要的诊断工具。通过查看explian()的输出信息，可以知道查询使用了那个索引，以及如果使用他的。对于任意查询来，都可以在最后添加一个explain()调用，但是调用explain()的时间必须是最后。

例如：

> ```
> db.carLicence.find({city:'Beijing'}).explain()
>
> //返回结果如下:
> {
>   "cursor" : "BasicCursor",
>   "n" : 1563247,
>   "nscannedObjects" : 96539732,
>   "nscanned" : 96539732,
>   "indexOnly" : false,
>   "millis" : 156, // 哭！
> }
> ```

说明：

- `cursor` - 索引项
- `n` - 查询结果的条目总数
- `nscanned` - 扫描并读取的索引条目数(index)
- `nscannedObjects` - 扫描并读取的全文条目数(documents)
- `indexOnly` - 是否使用了covered indexes功能，宝宝不哭，后面有详述。
- `millis` - 查询用时，单位是毫秒。