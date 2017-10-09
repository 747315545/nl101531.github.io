---
title: redis学习记录(四)-SpringDataRedis分析
tags:
  - redis
categories: redis
date: 2017-03-29 18:50:00
updated:  2017-03-29 18:50:00
---


# redis学习记录(四)-SpringDataRedis分析

标签（空格分隔）： redis

---

[Redis学习记录(一)--入门知识][1]
[Redis学习记录(二)--使用Jedis连接][2]
[redis学习记录(三)-redis中的数据结构][3]

### 1.简介
Spring Data Redis是对redis客户端(如jedis)的高度封装,支持多种客户端,因其高抽象,所以在某一个客户端不支持更新的时候可以容易切换到其他客户端.

本文是在Spring boot 1.5.2版本下测试.

需要引入架包
```xml

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!--spring boot start-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

### 2.配置
在Spring Boot下默认使用jedis作为客户端,并在包`org.springframework.boot.autoconfigure.data.redis`下,提供自动配置类`RedisProperties`,`RedisAutoConfiguration`等.

根据`RedisProperties`可以定位到可配置的属性,如:
``` properties
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=115.159.185.14
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=2000
```
在application.properties中配置即可,另外还有`Sentinel`和`Cluster`说明支持分布式和集群,博主研究不多就不瞎说这个了.

自动配置主要在`RedisAutoConfiguration`中,该类会提供三个bean:
1. JedisConnectionFactory : jedis连接控制工厂
2. RedisTemplate<Object, Object> : redis操作入口
3. StringRedisTemplate : redis操作入口

那么就开始入口学习.

----------
### 3.RedisTemplate<K, V>

RedisTemplate是操作的入口.该类继承了`RedisAccessor`,可以通过其拿到redis连接,实现了`RedisOperations`接口,获得了操作redis的能力,如下图所示:
![](http://ac-HSNl7zbI.clouddn.com/rUB5pG7qryosXsqkMNQ1u52FgHMVMwAX7OeVM3jy.jpg)

#### 3.1 Test case
那么具体操作过程是怎么样子的呢?写一个简单的测试去跟踪代码,如下代码,往redis中设置key为ping的字串.
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = Application.class)
public class RedisConnectTest {
  @Resource
  private RedisTemplate<String,String> redisTemplate;

  @Test
  public void testSetAndGet() {
    redisTemplate.opsForValue().set("ping","pong");
    System.out.println(redisTemplate.opsForValue().get("ping"));
  }
}
```
运行之后查看redis数据库,你会发现很奇怪的事情,如下图,代码中存入的是ping,但是到redis中后却是一堆字符+ping,这个原因是什么呢?接着跟踪代码.
![](http://ac-HSNl7zbI.clouddn.com/9O9oRCxhlph8oRYL6YirrY192jaYIOHAlGXAUemJ.jpg)

#### 3.2 XXXOperations<K, V>
上述代码的第一步先获取到了`ValueOperations`,在`RedisTemplate`中同样还有其他`XXXOperations`,根据官方文档,这些接口是针对redis的每一种命令的操作.如下表:

 | 接口 | 操作 | 
|:-----|:-----|
| ValueOperations | Redis string (or value) operations  |
| ListOperations | Redis list operations  |
| SetOperations | Redis set operations  |
| ZSetOperations | Redis zset (or sorted set) operations  |
| HashOperations | Redis hash operations  |
| HyperLogLogOperations | Redis HyperLogLog operations like (pfadd, pfcount,…​) |
| GeoOperations | Redis geospatial operations like GEOADD, GEORADIUS,…​) |
| BoundValueOperations | Redis string (or value) key bound operations |
| BoundListOperations | Redis list key bound operations |
| BoundSetOperations | Redis set key bound operations |
| BoundZSetOperations | Redis zset (or sorted set) key bound operations |
| BoundHashOperations | Redis hash key bound operations |
| BoundGeoOperations | Redis key bound geospatial operations. |

其中`BoundXXXOperations`是在key已知的情况下使用,其所有操作都是建立在有一个`certain key`的前提.可以看下源码就能明白了.

#### 3.3 XXXSerializer
那测试代码中第一步是获取了string类型的redis操作入口,然后执行set方法设置键和值,接着分析set方法.

```java
	public void set(K key, V value) {
		final byte[] rawValue = rawValue(value);
		execute(new ValueDeserializingRedisCallback(key) {

			protected byte[] inRedis(byte[] rawKey, RedisConnection connection) {
				connection.set(rawKey, rawValue);
				return null;
			}
		}, true);
	}
```
可以发现`rawKey()`方法和`rawValue()`方法对key和value进行了一次序列化操作.该序列化使用的类为RedisTemplate中的`XXXSerializer`,那么回到RedisTemplate,在`afterPropertiesSet()`方法中有以下初始化方法,默认使用的序列化方式为`JdkSerializationRedisSerializer`,也就是ObjectInputStream和ObjectOutputStream写入和读取.这也是写入到redis中却在redis数据库通过"ping"访问不到的原因.
```java
if (defaultSerializer == null) {

			defaultSerializer = new JdkSerializationRedisSerializer(
					classLoader != null ? classLoader : this.getClass().getClassLoader());
		}
		if (enableDefaultSerializer) {
			if (keySerializer == null) {
				keySerializer = defaultSerializer;
				defaultUsed = true;
			}
			if (valueSerializer == null) {
				valueSerializer = defaultSerializer;
				defaultUsed = true;
			}
			if (hashKeySerializer == null) {
				hashKeySerializer = defaultSerializer;
				defaultUsed = true;
			}
			if (hashValueSerializer == null) {
				hashValueSerializer = defaultSerializer;
				defaultUsed = true;
			}
		}

```
那么SpringDataRedis支持哪些序列化呢?从官网可以看到:
StringRedisSerializer: string类型序列化,也是最常用的类型
JdkSerializationRedisSerializer: jdk默认序列化
OxmSerializer : xml格式
JacksonJsonRedisSerializer : json格式

通过手动注入RedisTemplate,更改所选择的序列化方式.另外Spring提供了最常使用的`StringRedisTemplate`,实现了`StringRedisSerializer`序列化方式.
```java
	public StringRedisTemplate() {
		RedisSerializer<String> stringSerializer = new StringRedisSerializer();
		setKeySerializer(stringSerializer);
		setValueSerializer(stringSerializer);
		setHashKeySerializer(stringSerializer);
		setHashValueSerializer(stringSerializer);
	}
```

更改成`StringRedisTemplate`,再次执行,正常了.
![](http://ac-HSNl7zbI.clouddn.com/3PAtzJjJHXquNpAVgWJI0OVh8pJWDhVEl3FbD571.jpg)

#### 3.4 总结过程
1. 获取RedisTemplate
2. 获取操作入口XXXOperations
3. 使用RedisSerializer序列化key和value
4. 获取conn连接
4. 执行命令

### 4.发布与订阅
发布与订阅过程需要发布者,订阅者,以及把两者连在一起的桥梁.那么在SpringRedis中怎么实现呢?
订阅者:里面有一个处理方法即可.
```java
@Component
public class Listen {

  private static Logger logger = LoggerFactory.getLogger(Listen.class);

  private CountDownLatch latch = new CountDownLatch(1);

  public void handleMsg(String message) {
    logger.info("reciver msg :" + message);
    latch.countDown();
  }
  
  public CountDownLatch getLatch() {
    return latch;
  }
}
```
发布者:XXXRedisTemplate.convertAndSend(chanel,msg)即作为发布者存在.

连接桥梁:RedisMessageListenerContainer,该container监听Redis的消息,分发给各自的监听者.关键代码为

```java
@Configuration
public class PublishConfig {
  /**
   * 注入消息容器
   * @param jedisConnectionFactory jedis连接池
   * @param listenerAdapter 监听适配器
   * @return bean
   */
  @Bean
  public RedisMessageListenerContainer container(RedisConnectionFactory jedisConnectionFactory,
      MessageListenerAdapter listenerAdapter){
    RedisMessageListenerContainer container = new RedisMessageListenerContainer();
    container.setConnectionFactory(jedisConnectionFactory);
    //绑定监听者与信道的管理
    container.addMessageListener(listenerAdapter,new PatternTopic("java"));
    return container;
  }

  @Bean
  public MessageListenerAdapter adapter(Listen listen){
    //指定监听者和监听方法
    return new MessageListenerAdapter(listen,"handleMsg");
  }
}
```

测试:
```java
  @Test
  public void testPublish() throws InterruptedException {
    stringRedisTemplate.convertAndSend("java","hello world");
    listen.getLatch().await();
  }
```
![](http://ac-HSNl7zbI.clouddn.com/yhidqhoWBD7Un7XLH6WQYjIEl82Ve0R2jzCEzMrn.jpg)

### 5.事务
对于事务的操作是通过SessionCallback实现,该接口保证其内部所有操作都是在同一个Session中的,在最后exec的时候执行全部操作.关键代码如下
```java
    RedisConnectionUtils.bindConnection(factory, enableTransactionSupport);
    execute(this)
```
```java
 @Test
  public void testMulti() {
    boolean isThrow = false;
    List<Object> result = null;
    try {
      result = stringRedisTemplate.execute(new SessionCallback<List<Object>>() {
        @Override
        public List<Object> execute(RedisOperations operations) throws
            DataAccessException {
          operations.multi();
          ValueOperations<String,String> ops = operations.opsForValue();
          ops.set("ping1","pong1");
          ops.set("ping2","pong2");
          if (isThrow){
            throw new IllegalArgumentException("测试异常");
          }
          return operations.exec();
        }
      });
    } catch (Exception e) {
      e.printStackTrace();
    }

    System.out.println(result);
  }
```

### 6.管道
直接引用官方案例
```java
//pop a specified number of items from a queue
List<Object> results = stringRedisTemplate.executePipelined(
  new RedisCallback<Object>() {
    public Object doInRedis(RedisConnection connection) throws DataAccessException {
      StringRedisConnection stringRedisConn = (StringRedisConnection)connection;
      for(int i=0; i< batchSize; i++) {
        stringRedisConn.rPop("myqueue");
      }
    return null;
  }
});
```

还有脚本执行等,在官方文档中都有案例,这里就不复制粘贴了,如有错误请指出,不胜感激.


参考文档:

http://docs.spring.io/spring-data/redis/docs/1.8.1.RELEASE/reference/html/#redis:template

github:

https://github.com/nl101531/JavaWEB


  [1]: http://www.jianshu.com/p/da69edda2a43
  [2]: http://www.jianshu.com/p/c3b8180af944p/da69edda2a43
  [3]: http://mrdear.cn/2017/03/26/linux/redis%E5%AD%A6%E4%B9%A0%E8%AE%B0%E5%BD%95%28%E4%B8%89%29-redis%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/