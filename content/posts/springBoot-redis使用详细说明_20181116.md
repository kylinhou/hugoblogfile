---
title: Spring Data Redis使用详细介绍
categories: ["springboot"]
tags: ["springboot", "redis"]
date: 2018-04-23
author: "kylin"
grammar_cjkRuby: true
---
# Spring Data Redis使用详细介绍

## redis

redis是一种特殊类型的数据库，称之为key-value存储。说到key-value存储，可能会想到Map。不太夸张的说，它就是一个持久化的哈希Map。

## 连接到redis

既然要使用redis，首先要考虑如何连接redis。redis连接工厂会生成到redis服务器的连接。Spring Data Redis为四种redis客户端实现了连接工厂：

* JedisConnectionFactory
* JredisConnectionFactory
* LettuceConnectionFactory
* SrpConnectionFactory

具体使用哪一种客户端取决于你。

接下来将连接工厂配置为Spring的bean，以JedisConnectionFactory为例：

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisConnectionFactory redisCF() {
        return new JedisConnectionFactory();
    }
}
```

这样就创建好一个JedisConnectionFactory工厂了，现在就可以通过这个工厂获取redis的连接。测试一下：

```java
@SpringBootTest(classes = SpringlearnApplication.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class RedisConfigTest {

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;
    
    @Test
    public void redisCF() {
        RedisConnection conn = redisConnectionFactory.getConnection();
        conn.set("hello".getBytes(), "world".getBytes());
        System.out.println(new String(conn.get("hello".getBytes())));
    }
}
```

运行报错：

```sh
Caused by: java.net.ConnectException: Connection refused: connect
	at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
	at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:85)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at redis.clients.jedis.Connection.connect(Connection.java:184)
```

现在报错并不是因为程序真的有问题，我们在创建工厂的时候，直接return new JedisConnectionFactory();使用默认的构造器创建的工厂，那么工厂会向本地的6379端口创建连接，并且没有使用密码。因为本地没有redis服务，所以连接出错。

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisConnectionFactory redisCF() {
        // Standalone配置
        RedisStandaloneConfiguration redisStandaloneConfiguration =
                new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName("10.101.94.173");
        redisStandaloneConfiguration.setDatabase(1);
//        redisStandaloneConfiguration.setPassword(RedisPassword.of("123456"));
        redisStandaloneConfiguration.setPort(1000);
        // 通过指定配置创建工厂
        JedisConnectionFactory jc = new JedisConnectionFactory(redisStandaloneConfiguration);
        return jc;
    }
}
```

这次在工厂中使用了RedisStandaloneConfiguration，然后配置了ip、port、db等信息，如果需要你可以配置密码。

> `JedisConnectionFacotory`从`Spring Data Redis 2.0`开始已经不推荐直接显示设置连接的信息了，一方面为了使配置信息与建立连接工厂解耦，另一方面抽象出`Standalone`,`Sentinel`和`RedisCluster`三种模式的环境配置类和一个统一的jedis客户端连接配置类（用于配置连接池和SSL连接），使得我们可以更加灵活方便根据实际业务场景需要来配置连接信息。

再次进行测试，成功。现在已经可以成功向 10.101.94.173:1000这个redis上读写数据了。

上面说过有多种连接工厂可选择，使用方式都一样,下面是LettuceConnectionFactory的使用：

```java
    @Bean
    public RedisConnectionFactory redisCFLettuce() {
        // Standalone配置
        RedisStandaloneConfiguration redisStandaloneConfiguration =
                new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName("10.101.94.173");
        redisStandaloneConfiguration.setDatabase(1);
//        redisStandaloneConfiguration.setPassword(RedisPassword.of("123456"));
        redisStandaloneConfiguration.setPort(1000);
        // 通过指定配置创建工厂
        LettuceConnectionFactory jc = new LettuceConnectionFactory(redisStandaloneConfiguration);
        return jc;
    }
```

上面进行redis写操作我们使用的方式：conn.set("hello".getBytes(), "world".getBytes());

读操作：conn.get("hello".getBytes())

操作的都是字节数组，直接操作字节数组非常不方便。幸好，Spring Data Redis以模版的形式提供了较高级的数据访问方式。Spring Data Redis 提供了两个模版：

* RedisTemplate
* StringRedisTemplate

下面介绍下如何使用RedisTemplate

## 使用RedisTemplate

RedisTemplate可以极大的简化Redis数据访问，能够让我们持久化各种类型的key和value，并不局限于字节数组。 RedisTemplate主要支持String，List,Hash,Set,ZSet这几种方式的参数，其对应的方法分别是opsForValue()、opsForList()、opsForHash()、opsForSet()、opsForZSet()。

因为很大部分的key 和value都是String类型的，所以StringRedisTemplate扩展了RedisTemplate，只关注String类型。

首先我们先创建一个RedisTemplate模板，类型的key是String类型，value是Object类型（如果key和value都是String类型，建议使用StringRedisTemplate）

```java
    @Bean
    public RedisTemplate redisTemplate(@Qualifier("jedisConnectionFactory")RedisConnectionFactory factory){
        //创建一个模板类
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        //将刚才的redis连接工厂设置到模板类中
        template.setConnectionFactory(factory);
        return template;
    }
```

测试一下RedisTemplate能否正常操作redis

```java
    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 使用 RedisTemplate 进行redis操作
     */
    @Test
    public void redisTemplateTest() {
        redisTemplate.opsForValue().set("key1", "value1");
        System.out.println(redisTemplate.opsForValue().get("key1"));
    }
```

成功！

操作一下集合

```java
@Test
    public void testRedisTemplateList(){
    
        Pruduct prud  = new Pruduct(1, "洗发水", "100ml");
        Pruduct prud2  = new Pruduct(2, "洗面奶", "200ml");
        //依次从尾部添加元素
        template.opsForList().rightPush("pruduct", prud);
        template.opsForList().rightPush("pruduct", prud);
        //查询索引0到商品总数-1索引（也就是查出所有的商品）
        List<Object> prodList = template.opsForList().range("pruduct", 0,template.opsForList().size("pruduct")-1);
        for(Object obj:prodList){
            System.out.println((Pruduct)obj);
        }
        System.out.println("产品数量:"+template.opsForList().size("pruduct"));
        
    }
```

## 使用 key 和 value的序列化器

### 序列化与反序列化的概念

这里简单说一下什么是序列化：

* 把对象转换为字节序列的过程称为对象的序列化

* 把字节序列恢复为对象的过程称为对象的反序列化。

    对象的序列化主要有两种用途：
    　　1） 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；
    　　2） 在网络上传送对象的字节序列。

　　在很多应用中，需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便长期保存。比如最常见的是Web服务器中的Session对象，当有 10万用户并发访问，就有可能出现10万个Session对象，内存可能吃不消，于是Web容器就会把一些seesion先序列化到硬盘中，等要用了，再把保存在硬盘中的对象还原到内存中。

　　当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个Java对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为Java对象。

### redis 中的序列化

当某个条目保存到Redis key-value存储的时候，key和value都会使用redis的序列化器进行序列化。Spring Redis Data提供了多个这样的序列化器：

* GenericToStringSerializer：使用Spring转换服务进行序列化；

* JacksonJsonRedisSerializer：使用Jackson 1，将对象序列化为JSON；

* Jackson2JsonRedisSerializer：使用Jackson 2，将对象序列化为JSON；

* JdkSerializationRedisSerializer：使用Java序列化；

* OxmSerializer：使用Spring O/X映射的编排器和解排器（marshaler和unmarshaler）实

  现序列化，用于XML序列化；

* StringRedisSerializer：序列化String类型的key和value。

RedisTemplate会使用 JdkSerializationRedisSerializer ，这意味着key和value都是通过jdk进行序列化。

StringRedisTemplate 默认会使用 StringRedisSerializer。

这些默认的设置适合很多的场景，但有些场景可能需要用不同的序列化器。假设我们使用RedisTemplate的时候希望key是String类型，value序列化为json，那么就需要如下所示：



使用默认的RedisTemplate 存储String类型：

```java
redisTemplate.opsForValue().set("key1", "value1");
```

在redis中的操作：

> "SET" "\xac\xed\x00\x05t\x00\x04key1" "\xac\xed\x00\x05t\x00\x06value1"

key和value都是通过jdk进行序列化

使用默认的RedisTemplate 存储object对象：

```java
        Product product = new Product();
        product.setId(1);
        product.setName("test");
        redisTemplate.opsForValue().set("key2", product);
```

redis中的操作：

> "SET" "\xac\xed\x00\x05t\x00\x04key2" "\xac\xed\x00\x05sr\x00\x1bcom.kylin.chapter12.Product\xc7\xda\x10\xdf\xa7\xce\xc6\xe6\x02\x00\x02I\x00\x02idL\x00\x04namet\x00\x12Ljava/lang/String;xp\x00\x00\x00\x01t\x00\x04test"

下面将value用json的序列化方式进行存储。

```java
    @Bean(name="redisTemplateJson")
    public RedisTemplate redisTemplateJson(@Qualifier("jedisConnectionFactory")RedisConnectionFactory factory){
        //创建一个模板类
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        //将刚才的redis连接工厂设置到模板类中
        template.setConnectionFactory(factory);
        // 设置key的序列化器
        template.setKeySerializer(new StringRedisSerializer());
        // 设置value的序列化器
        //使用Jackson 2，将对象序列化为JSON
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        //json转对象类，不设置默认的会将json转成hashmap
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }
```

```java
    @Test
    public void redisTemplateTest() {
        Product product = new Product();
        product.setId(1);
        product.setName("test");
        redisTemplate.opsForValue().set("key2", product);
    }
```

redis中的操作：

> "SET" "key1" "\"value1\""

> "SET" "key2" "[\"com.kylin.chapter12.Product\",{\"id\":1,\"name\":\"test\"}]"

为什么保存的value中带有了类信息，可以只保存数据本身吗？试一试

在存储对象时如果先进行json转换：

```java
redisTemplate.opsForValue().set("key2", JSONObject.toJSONString(product));
```

redis中保存的：

> "SET" "key2" "\"{\\\"id\\\":1,\\\"name\\\":\\\"test\\\"}\""

这样保存的数据就看着不舒服了，多层转义。看来也不是这样。

## 使用pipeline

### 为什么使用pipeline

* 减少RTT（Round Time Trip）
* 减少系统IO的调用

**允许客户端可以一次发送多条命令，而不等待上一条命令执行的结果，减少了调用次数、减少了IO操作。IO调用涉及到用户态到内核态之间的切换**。

Pipeline在某些场景下非常有用，比如有多个command需要被“**及时的**”提交，而且他们对相应结果没有互相依赖，对结果响应也无需立即获得，那么pipeline就可以充当这种“**批处理**”的工具；而且在一定程度上，可以较大的提升性能，性能提升的原因**主要是TCP连接中减少了“交互往返”的时间**。

### pipeline的实现原理

要支持Pipeline，其实既要服务端的支持，也要客户端支持。对于服务端来说，所需要的是能够处理一个客户端通过同一个TCP连接发来的多个命令，可以理解为，这里将多个命令切分，和处理单个命令一样（之前老生常谈的黏包现象），Redis就是这样处理的。而客户端，则是要将多个命令缓存起来，缓冲区满了就发送，然后再写缓冲，最后才处理Redis的应答，如Jedis。

### 注意点

Redis的Pipeline和Transaction不同，Transaction会存储客户端的命令，最后一次性执行，而Pipeline则是处理一条，响应一条，但是这里却有一点，就是客户端会并不会调用read去读取socket里面的缓冲数据，这也就造就了，如果Redis应答的数据填满了该接收缓冲（SO_RECVBUF），那么客户端会通过ACK，WIN=0（接收窗口）来控制服务端不能再发送数据，那样子，数据就会缓冲在Redis的客户端应答列表里面。所以需要**注意控制Pipeline的大小**。



### 单节点下使用pipeline

获取一个Pipeline

```java
    @Bean
    public  Pipeline getPipeline() {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxIdle(maxIdl);
        config.setMaxTotal(500);
        config.setMaxWaitMillis(3 * 100000000);
        config.setTestOnBorrow(true);
        config.setTestOnReturn(true);

        JedisPool jedisPool = new JedisPool(config, hostName, port,timeout);
        Jedis rds = jedisPool.getResource();
        Pipeline pp = rds.pipelined();
        return pp;
    }
```

获取到Pipeline之后，就可以使用了

```java
@Test
    public void testPipeline() {
        pipeline.select(1);///选择db
        int id = 1;
        for (int i = 1; i <= 10; i++) {
            String key = "test"+id;
            String value = "value"+id;
            try {
                ////进行key value操作
//                pipeline.set(key.getBytes(), value.getBytes());
//                pipeline.hset(key,"summary",value);
//                pipeline.del(key);
                pipeline.set(key,value);

                if (i % 10 == 0) {
                    pipeline.sync();
                    pipeline.clear();
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
            }
            id++;
        }
    }
```

成功运行，效率大大提升！

### cluster集群下使用pipeline

JedisClusterConfig:

```java
package com.kylin.chapter12;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import redis.clients.jedis.HostAndPort;
import redis.clients.jedis.JedisCluster;
import redis.clients.jedis.JedisPoolConfig;

import java.util.HashSet;
import java.util.Set;

@Configuration
public class JedisClusterConfig {

    @Value("${spring.redis.cluster.nodes}")
    private String clusterNodes;
    @Value("${spring.redis.timeout}")
    private int timeout;
    @Value("${spring.redis.jedis.pool.max-idle}")
    private int maxIdle;
    @Value("${spring.redis.jedis.pool.max-wait}")
    private long maxWaitMillis;
    @Value("${spring.redis.commandTimeout}")
    private int commandTimeout;
    @Value("${spring.redis.password}")
    private String password;

    @Bean
    public JedisCluster getJedisCluster(){
        String[] serverArray = clusterNodes.split(",");
        Set<HostAndPort> nodes = new HashSet<>();
        for (String ipPort : serverArray) {
            String[] ipPortPair = ipPort.split(":");
            nodes.add(new HostAndPort(ipPortPair[0].trim(), Integer.valueOf(ipPortPair[1].trim())));
        }
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxIdle(maxIdle);
        jedisPoolConfig.setMaxWaitMillis(maxWaitMillis);
//        JedisCluster jedisCluster=new JedisCluster(nodes,commandTimeout,timeout,maxIdle,password,jedisPoolConfig);
        JedisCluster jedisCluster=new JedisCluster(nodes,jedisPoolConfig);
        return jedisCluster;
    }
}

```

JedisClusterPipeline 工具类：

```java
/**
 * Copyright: Copyright (c) 2015
 *
 * @author youaremoon
 * @date 2016年6月25日
 * @version V1.0
 */
package com.kylin.chapter12;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import redis.clients.jedis.*;
import redis.clients.jedis.exceptions.JedisMovedDataException;
import redis.clients.jedis.exceptions.JedisRedirectionException;
import redis.clients.util.JedisClusterCRC16;
import redis.clients.util.SafeEncoder;

import java.io.Closeable;
import java.lang.reflect.Field;
import java.util.*;

/**
 * 在集群模式下提供批量操作的功能。 <br/>
 * 由于集群模式存在节点的动态添加删除，且client不能实时感知（只有在执行命令时才可能知道集群发生变更），
 * 因此，该实现不保证一定成功，建议在批量操作之前调用 refreshCluster() 方法重新获取集群信息。<br />
 * 应用需要保证不论成功还是失败都会调用close() 方法，否则可能会造成泄露。<br/>
 * 如果失败需要应用自己去重试，因此每个批次执行的命令数量需要控制。防止失败后重试的数量过多。<br />
 * 基于以上说明，建议在集群环境较稳定（增减节点不会过于频繁）的情况下使用，且允许失败或有对应的重试策略。<br />
 *
 * 该类非线程安全
 *
 * @author youaremoon
 * @version
 * @since Ver 1.1
 */

public class JedisClusterPipeline extends PipelineBase implements Closeable {
    private static final Logger LOGGER = LoggerFactory.getLogger(JedisClusterPipeline.class);

    // 部分字段没有对应的获取方法，只能采用反射来做
    // 你也可以去继承JedisCluster和JedisSlotBasedConnectionHandler来提供访问接口
    private static final Field FIELD_CONNECTION_HANDLER;
    private static final Field FIELD_CACHE;
    static {
        FIELD_CONNECTION_HANDLER = getField(BinaryJedisCluster.class, "connectionHandler");
        FIELD_CACHE = getField(JedisClusterConnectionHandler.class, "cache");
    }

    private JedisSlotBasedConnectionHandler connectionHandler;
    private JedisClusterInfoCache clusterInfoCache;
    private Queue<Client> clients = new LinkedList<Client>();	// 根据顺序存储每个命令对应的Client
    private Map<JedisPool, Jedis> jedisMap = new HashMap<>();	// 用于缓存连接
    private boolean hasDataInBuf = false;	// 是否有数据在缓存区

    /**
     * 根据jedisCluster实例生成对应的JedisClusterPipeline
     * @param
     * @return
     */
    public static JedisClusterPipeline pipelined(redis.clients.jedis.JedisCluster jedisCluster) {
        JedisClusterPipeline pipeline = new JedisClusterPipeline();
        pipeline.setJedisCluster(jedisCluster);
        return pipeline;
    }

    public JedisClusterPipeline() {
    }

    public void setJedisCluster(redis.clients.jedis.JedisCluster jedis) {
        connectionHandler = getValue(jedis, FIELD_CONNECTION_HANDLER);
        clusterInfoCache = getValue(connectionHandler, FIELD_CACHE);
    }

    /**
     * 刷新集群信息，当集群信息发生变更时调用
     * @param
     * @return
     */
    public void refreshCluster() {
        connectionHandler.renewSlotCache();
    }

    /**
     * 同步读取所有数据. 与syncAndReturnAll()相比，sync()只是没有对数据做反序列化
     */
    public void sync() {
        innerSync(null);
    }

    /**
     * 同步读取所有数据 并按命令顺序返回一个列表
     *
     * @return 按照命令的顺序返回所有的数据
     */
    public List<Object> syncAndReturnAll() {
        List<Object> responseList = new ArrayList<Object>();

        innerSync(responseList);

        return responseList;
    }

    private void innerSync(List<Object> formatted) {
        HashSet<Client> clientSet = new HashSet<Client>();

        try {
            for (Client client : clients) {
                // 在sync()调用时其实是不需要解析结果数据的，但是如果不调用get方法，发生了JedisMovedDataException这样的错误应用是不知道的，因此需要调用get()来触发错误。
                // 其实如果Response的data属性可以直接获取，可以省掉解析数据的时间，然而它并没有提供对应方法，要获取data属性就得用反射，不想再反射了，所以就这样了
                Object data = generateResponse(client.getOne()).get();
                if (null != formatted) {
                    formatted.add(data);
                }

                // size相同说明所有的client都已经添加，就不用再调用add方法了
                if (clientSet.size() != jedisMap.size()) {
                    clientSet.add(client);
                }
            }
        } catch (JedisRedirectionException jre) {
            if (jre instanceof JedisMovedDataException) {
                // if MOVED redirection occurred, rebuilds cluster's slot cache,
                // recommended by Redis cluster specification
                refreshCluster();
            }

            throw jre;
        } finally {
            if (clientSet.size() != jedisMap.size()) {
                // 所有还没有执行过的client要保证执行(flush)，防止放回连接池后后面的命令被污染
                for (Jedis jedis : jedisMap.values()) {
                    if (clientSet.contains(jedis.getClient())) {
                        continue;
                    }

                    flushCachedData(jedis);
                }
            }

            hasDataInBuf = false;
            close();
        }
    }

    @Override
    public void close() {
        clean();

        clients.clear();

        for (Jedis jedis : jedisMap.values()) {
            if (hasDataInBuf) {
                flushCachedData(jedis);
            }

            jedis.close();
        }

        jedisMap.clear();

        hasDataInBuf = false;
    }

    private void flushCachedData(Jedis jedis) {
        try {
            jedis.getClient().getAll();
        } catch (RuntimeException ex) {
            // 其中一个client出问题，后面出问题的几率较大
        }
    }

    @Override
    protected Client getClient(String key) {
        byte[] bKey = SafeEncoder.encode(key);

        return getClient(bKey);
    }

    @Override
    protected Client getClient(byte[] key) {
        Jedis jedis = getJedis(JedisClusterCRC16.getSlot(key));

        Client client = jedis.getClient();
        clients.add(client);

        return client;
    }

    private Jedis getJedis(int slot) {
        JedisPool pool = clusterInfoCache.getSlotPool(slot);

        // 根据pool从缓存中获取Jedis
        Jedis jedis = jedisMap.get(pool);
        if (null == jedis) {
            jedis = pool.getResource();
            jedisMap.put(pool, jedis);
        }

        hasDataInBuf = true;
        return jedis;
    }

    private static Field getField(Class<?> cls, String fieldName) {
        try {
            Field field = cls.getDeclaredField(fieldName);
            field.setAccessible(true);

            return field;
        } catch (NoSuchFieldException | SecurityException e) {
            throw new RuntimeException("cannot find or access field '" + fieldName + "' from " + cls.getName(), e);
        }
    }

    @SuppressWarnings({"unchecked" })
    private static <T> T getValue(Object obj, Field field) {
        try {
            return (T)field.get(obj);
        } catch (IllegalArgumentException | IllegalAccessException e) {
            LOGGER.error("get value fail", e);

            throw new RuntimeException(e);
        }
    }
}
```

测试一下：

```java
 @Test
    public void testRedisCluster() {
        JedisClusterPipeline pipeline = JedisClusterPipeline.pipelined(jedisCluster);
        pipeline.refreshCluster();
        List<Object> batchResult = null;
        long s = System.currentTimeMillis();
        try {

            // batch write
            for (int i = 1; i <= 10; i++) {
                String key = "battle:info:"+i;
                int userid1 = 20000000+i;
                int userid2 = 20000000+i+1;
                String value = "123";
                pipeline.set(key, value);
                if (i % 10 == 0) {
                    pipeline.sync();
                }
            }
            batchResult = pipeline.syncAndReturnAll();
        } finally {
            pipeline.close();
//            jedisCluster.close();
        }
    }
```

## Spring Data Redis 与 Jedis 的关系

- jedis

> [jedis](https://github.com/xetorthio/jedis)是redis的java客户端，通过它可以对redis进行操作。与之功能相似的还包括：[Lettuce](https://github.com/lettuce-io/lettuce-core/)等

- spring-data-redis

> 它依赖jedis或Lettuce，实际上是对jedis这些客户端的封装，提供一套与客户端无关的api供应用使用，从而你在从一个redis客户端切换为另一个客户端，不需要修改业务代码。

**Spring-data-redis**为spring-data模块中对redis的支持部分，简称为“SDR”，提供了基于jedis客户端API的高度封装以及与spring容器的整合，事实上jedis客户端已经足够简单和轻量级，而spring-data-redis反而具有“过度设计”的嫌疑。

### jedis的不足

​    jedis客户端在编程实施方面存在如下不足：

​    1) connection管理缺乏自动化，connection-pool的设计缺少必要的容器支持。

​    2) 数据操作需要关注“序列化”/“反序列化”，因为jedis的客户端API接受的数据类型为string和byte，对结构化数据(json,xml,pojo等)操作需要额外的支持。

​    3) 事务操作纯粹为硬编码

​    4) pub/sub功能，缺乏必要的设计模式支持，对于开发者而言需要关注的太多。

​    不过jedis与spring整合，也是非常的简单，参见“[jedis连接池实例](http://shift-alt-ctrl.iteye.com/blog/1885910)”.

**一.  spring-data-redis针对jedis提供了如下功能**：

​    **1**. 连接池自动管理，提供了一个高度封装的“RedisTemplate”类

​    **2**. 针对jedis客户端中大量api进行了归类封装,将同一类型操作封装为operation接口

- ValueOperations：简单K-V操作
- SetOperations：set类型数据操作
- ZSetOperations：zset类型数据操作
- HashOperations：针对map类型的数据操作
- ListOperations：针对list类型的数据操作

​    **3**. 提供了对key的“bound”(绑定)便捷化操作API，可以通过bound封装指定的key，然后进行一系列的操作而无须“显式”的再次指定Key，即BoundKeyOperations：

- BoundValueOperations
- BoundSetOperations
- BoundListOperations
- BoundSetOperations
- BoundHashOperations

​    **4**. 将事务操作封装，有容器控制。

​    **5**. 针对数据的“序列化/反序列化”，提供了多种可选择策略(RedisSerializer)

- JdkSerializationRedisSerializer：POJO对象的存取场景，使用JDK本身序列化机制，将pojo类通过ObjectInputStream/ObjectOutputStream进行序列化操作，最终redis-server中将存储字节序列。是目前最常用的序列化策略。
- StringRedisSerializer：Key或者value为字符串的场景，根据指定的charset对数据的字节序列编码成string，是“new String(bytes, charset)”和“string.getBytes(charset)”的直接封装。是最轻量级和高效的策略。
- JacksonJsonRedisSerializer：jackson-json工具提供了javabean与json之间的转换能力，可以将pojo实例序列化成json格式存储在redis中，也可以将json格式的数据转换成pojo实例。因为jackson工具在序列化和反序列化时，需要明确指定Class类型，因此此策略封装起来稍微复杂。【需要jackson-mapper-asl工具支持】
- OxmSerializer：提供了将javabean与xml之间的转换能力，目前可用的三方支持包括jaxb，apache-xmlbeans；redis存储的数据将是xml工具。不过使用此策略，编程将会有些难度，而且效率最低；不建议使用。【需要spring-oxm模块的支持】

​    针对“序列化和发序列化”中JdkSerializationRedisSerializer和StringRedisSerializer是最基础的策略，原则上，我们可以将数据存储为任何格式以便应用程序存取和解析(其中应用包括app，hadoop等其他工具)，不过在设计时仍然不推荐直接使用“JacksonJsonRedisSerializer”和“OxmSerializer”，因为无论是json还是xml，他们本身仍然是String。

​    如果你的数据需要被第三方工具解析，那么数据应该使用StringRedisSerializer而不是JdkSerializationRedisSerializer。

​    如果你的数据格式必须为json或者xml，那么在编程级别，在redisTemplate配置中仍然使用StringRedisSerializer，在存储之前或者读取之后，使用“SerializationUtils”工具转换转换成json或者xml，请参见下文实例。

​    **6**. 基于设计模式，和JMS开发思路，将pub/sub的API设计进行了封装，使开发更加便捷。

### Spring data redis的不足

spring-data-redis中，并没有对sharding提供良好的封装，如果你的架构是基于sharding，那么你需要自己去实现，这也是sdr和jedis相比，唯一缺少的特性。

## 在Spring boot中的使用

在spring boot中redis连接池JedisConnectionFactory和redis模板类(RedisTemplate和StringRedisTemplate)。直接在应用中通过@Autowire就可以注入以上对象。

所以在spring boot中使用redis跟其他组件一样非常简单，只需要确保在配置文件中进行了相关配置，就可以直接获取bean进行使用：

```properties
#配置缓存redis
spring.redis.database=1
# Redis服务器地址
spring.redis.host=10.101.94.173
# Redis服务器连接端口
spring.redis.port=1000
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）ms
spring.redis.jedis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.keytimeout=1000
spring.redis.timeout=0
```

其他什么都不用操作，可以直接测试一下能否连接redis进行处理：

```java
package com.kylin.chapter12;

import com.alibaba.fastjson.JSONObject;
import com.kylin.SpringlearnApplication;
import net.minidev.json.JSONArray;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.dao.DataAccessException;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnection;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import static org.junit.Assert.*;

@SpringBootTest(classes = SpringlearnApplication.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class RedisConfigTest {

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 直接使用 RedisConnection 进行redis操作
     */
    @Test
    public void redisCF() {
        RedisConnection conn = redisConnectionFactory.getConnection();
        conn.set("hello".getBytes(), "world".getBytes());
        System.out.println(conn);
        System.out.println(new String(conn.get("hello".getBytes())));
    }

    /**
     * 使用 RedisTemplate 进行redis操作
     */
    @Test
    public void redisTemplateTest() {
        Product product = new Product();
        product.setId(1);
        product.setName("test");
        redisTemplate.opsForValue().set("key2", JSONObject.toJSONString(product));
        System.out.println(redisTemplate.opsForValue().get("key2"));
    }

}
```

都可以成功运行！

当然，如果用户自己实现了相关bean，spring boot会使用用户自己的实现。

另外在spring boot中如果使用RedisTemplate进行redis的操作，单节点与集群是没有区别的，只是在配置文件中配置的ip不同：

单节点：spring.redis.host=10.101.94.173

集群：spring.redis.cluster.nodes=10.101.94.164:1002,10.101.94.163:1002,10.101.94.18:1002

## redis的其他知识

### Codis 