---
title: spring boot整合redis
categories: ["springboot"]
tags: ["springboot", "redis"]
date: 2018-04-23
author: "kylin"
grammar_cjkRuby: true
---
# spring boot整合redis

这篇文章介绍如何简单快速的使用redis。可以概括为下面几个步骤

1. 添加maven引用：spring-boot-starter-data-redis
2. 配置redis数据源
3. 创建redis的java配置类
4. 创建redis的使用工具类
5. 使用redis工具类进行redis操作

本文中的redis工具类相对比较简单，工具类的实现可以随意。

关于使用redis更为详细的说明，看这里：

## 添加redis引用

```java
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
			<version>1.5.6.RELEASE</version>
		</dependency>
```

## 配置文件配置redis源

```xml
##############################################################
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=172.25.34.5
#spring.redis.cluster.nodes=172.25.34.5:8005,172.25.34.5:8005,172.25.34.5:8005
# Redis服务器连接端口
spring.redis.port=1005
# Redis服务器连接密码（默认为空）
spring.redis.password=amazing@vivo=redis
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=0
##############################################################
```

注意：

* 连接redis单节点和cluster集群的配置是不一样的
* redis的密码设置

## 添加cache的配置类 

```java
package com.kylin.redis;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

import java.lang.reflect.Method;

/**
 * Created by 11041385 on 2018/8/4.
 */
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {
    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

    @SuppressWarnings("rawtypes")
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
        //设置缓存过期时间
        //rcm.setDefaultExpiration(60);//秒
        return rcm;
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}

```



## redis使用的工具类

```java
package com.kylin.redis;

/**
 * Created by 11041385 on 2018/8/4.
 */

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Component;

import java.io.Serializable;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * Created by lightClouds917
 * Date 2018/1/18
 * Description:操作redis的工具类
 */
@SuppressWarnings("unchecked")
@Component
public class RedisUtil {

    @SuppressWarnings("rawtypes")
    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 写入缓存
     *
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 写入缓存  指定时间  单位：秒
     *
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value, Long expireTime) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 判断缓存中是否有对应的value
     *
     * @param key
     * @return
     */
    public boolean exists(final String key) {
        return redisTemplate.hasKey(key);
    }

    /**
     * 读取缓存
     *
     * @param key
     * @return
     */
    public Object get(final String key) {
        Object result = null;
        ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
        result = operations.get(key);
        return result;
    }

    /**
     * 删除对应的value
     *
     * @param key
     */
    public void remove(final String key) {
        if (exists(key)) {
            redisTemplate.delete(key);
        }
    }

    /**
     * 批量删除key
     *
     * @param pattern
     */
    public void removePattern(final String pattern) {
        Set<Serializable> keys = redisTemplate.keys(pattern);
        if (keys.size() > 0) {
            redisTemplate.delete(keys);
        }
    }

    /**
     * 批量删除对应的value
     *
     * @param keys
     */
    public void remove(final String... keys) {
        for (String key : keys) {
            remove(key);
        }
    }
}


```

## 使用redis

### 手动进行redis操作

```java
    @RequestMapping("/getRedis")
    public String getDataFromRedis(HttpServletRequest request) {
        String id = request.getParameter("id");
        String data = (String)redisUtil.get(id);
        System.out.println(data);
        return data;
    }
```

使用redisUtil工具类进行redis的操作

### 使用注解进行redis操作

```java
   @RequestMapping("/getRedis")
    @Cacheable(value = "data",key="#request.getParameter(\"id\")")
    public String getDataFromRedis(HttpServletRequest request) {
        String id = request.getParameter("id");
        String data = "5"+id;
        System.out.println(data);
        return data;
    }
```

请求到来之后，首先会去缓存中查询key是否已经存在

```sh
"EXISTS" "1"
```

如果不存在，进入方法内部走流程，并且把key和value存入redis。

```sh
"SET" "1" "\"51\""
```

如果key已经存在，从redis中取出数据，返回

## 关于序列化

使用xshell 连接redis插入的数据 等同于使用StringRedisTemplate操作redis 

StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。

RedisTemplate默认采用的是JDK的序列化策略，保存的key和value都是采用此策略序列化保存的。

**当你的redis数据库里面本来存的是字符串数据或者你要存取的数据就是字符串类型数据的时候，那么你就使用StringRedisTemplate即可，但是如果你的数据是复杂的对象类型，而取出的时候又不想做任何的数据转换，直接从Redis里面取出一个对象，那么使用RedisTemplate是更好的选择。** 

### JdkSerializationRedisSerializer 

```java
@Test  
public void testJdkSerialiable() {  
    RedisTemplate<String, Serializable> redis = new RedisTemplate<String, Serializable>();  
    redis.setConnectionFactory(connectionFactory);  
    redis.setKeySerializer(ApplicationConfig.StringSerializer.INSTANCE);  
    redis.setValueSerializer(new JdkSerializationRedisSerializer());  
    redis.afterPropertiesSet();  
  
    ValueOperations<String, Serializable> ops = redis.opsForValue();  
  
    User user1 = new User();  
    user1.setUserName("user1");  
    user1.setAge(20);  
  
    String key1 = "users/user1";  
    User user11 = null;  
  
    long begin = System.currentTimeMillis();  
    for (int i = 0; i < 100; i++) {  
        ops.set(key1,user1);  
        user11 = (User)ops.get(key1);  
    }  
    long time = System.currentTimeMillis() - begin;  
    System.out.println("jdk time:"+time);  
    assertThat(user11.getUserName(),is("user1"));  
}  
```

### JacksonJsonRedisSerializer 

```java
@Test  
public void testJacksonSerialiable() {  
    RedisTemplate<String, Object> redis = new RedisTemplate<String, Object>();  
    redis.setConnectionFactory(connectionFactory);  
    redis.setKeySerializer(ApplicationConfig.StringSerializer.INSTANCE);  
    redis.setValueSerializer(new JacksonJsonRedisSerializer<User>(User.class));  
    redis.afterPropertiesSet();  
  
    ValueOperations<String, Object> ops = redis.opsForValue();  
  
    User user1 = new User();  
    user1.setUserName("user1");  
    user1.setAge(20);  
      
    User user11 = null;  
    String key1 = "json/user1";  
  
    long begin = System.currentTimeMillis();  
    for (int i = 0; i < 100; i++) {  
        ops.set(key1,user1);  
        user11 = (User)ops.get(key1);  
    }  
    long time = System.currentTimeMillis() - begin;  
  
    System.out.println("json time:"+time);  
    assertThat(user11.getUserName(),is("user1"));  
}  
```







