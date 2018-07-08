---
toc: true
title: Spring Data Redis
date: 2018-04-11 15:40:43
tags: [redis, spring data redis]
---


Redis是一种特殊类型的数据库，它被称为key-value存储。
Redis教程官网：http://www.redis.net.cn/tutorial/3501.html

Spring Data Redis是Spring Data系列的一部分，使用它可以轻松配置和访问Spring应用程序中的redis。如果直接操作redis，我们需要处理字节数组和各种类型之间的转换问题，类似于JDBC模板，Spring Data Redis提供了RedisTemplate。
<!--more-->
废话少说，咱们来实践一下。

### pom.xml
Spring Data Redis使用spring-data-redis，同时依赖jedis。
这里需要非常注意版本问题。
**spring-data-redis 2.0版本**
![Alt text](https://app.yinxiang.com/shard/s15/res/dcb7d832-be14-4db7-a12b-39368ebcaee9/1523431453714.png)
2.0以上版本要求 jdk 6以上， 以及Spring Framework 5.0.3以上。
**spring-data-redis 1.8版本**
![Alt text](https://app.yinxiang.com/shard/s15/res/c9984bec-cd00-4d89-965c-5d1fd3e2b6f3/1523431347709.png)
这里1.8版本要求jdk 6以上，以及spring framework4.3.15以上。


我一开始用的使用spring-data-redis 2.0.6，但是spring是4.3.13。所以装配Bean的时候一直报错。
```xml
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
            <version>1.8.11.RELEASE</version>
        </dependency>
```

### JavaConfig配置
Redis连接工厂会生成到Redis数据库服务器的连接。Spring Data Redis为四种Redis客户端实现提供了连接工厂：
- JedisConnectionFactory
- JredisConnectionFactory
- LettuceConnectionFactory
- SrpConnectionFactory

这里使用JedisConnectionFactory
```java
	/**
     * Redis连接工厂
     * @return
     */
    @Bean
    public RedisConnectionFactory redisConnectionFactory(){
        return new JedisConnectionFactory();
    }

    /**
     * Redis访问模板
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisTemplate<String, Spitter> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String, Spitter> redisTemplate = new RedisTemplate<String, Spitter>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        return redisTemplate;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory){
        return new StringRedisTemplate(redisConnectionFactory);
    }
```
Spring Data Redis以模板的形式提供了较高等级的数据访问方案	。实际上,Spring Data Redis提供了两个模板:
- RedisTemplate
- StringRedisTemplate

RedisTemplate<K, V>使用两个类型进行参数化。第一个是key类型，第二个是value类型。

如果你所使用的value和key都是String类型,那么可以考虑使用StringRedisTemplate。

### 使用RedisTemplate
![Alt text](https://app.yinxiang.com/shard/s15/res/bee56819-a3f9-478f-ad30-14bfbd333fbc/1523432112729.png)
对于每一种数据类型都有相应的方法，详细可以查阅源码或API文档有多少方法。
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {SpittrConfig.class})
public class ISpitterServiceTest {

    private static final org.slf4j.Logger LOG = LoggerFactory.getLogger(ISpitterServiceTest.class);

    @Autowired
    public RedisTemplate<String, Spitter> redisTemplate;

    @Test
    public void registerTest(){
        Spitter capljf = new Spitter();
        capljf.setUsername("capljf");
        capljf.setPassword("123456");
        // 操作单个值
        redisTemplate.opsForValue().set(capljf.getUsername(),capljf);
        String out = redisTemplate.opsForValue().get("capljf").toString();
        LOG.info("======================={}", out);
        // 操作List
        redisTemplate.opsForList().leftPush("spitterList",capljf);
        List<Spitter> spitterList = redisTemplate.opsForList().range("spitterList", 0, 2);
        LOG.info("======================={}", spitterList);
        // 操作Set
        redisTemplate.opsForSet().add("spitterSet", capljf);
        Spitter spitter = redisTemplate.opsForSet().randomMember("spitterSet");
        LOG.info("======================={}", spitter);
        redisTemplate.opsForSet().remove("spitterSet",capljf);
        spitter = redisTemplate.opsForSet().randomMember("spitterSet");
        LOG.info("======================={}", spitter);

    }

}
```



> 参考文献
> [1] 《Spring实战》4th
> [2] [Spring Data Redis](https://docs.spring.io/spring-data/redis/docs/1.8.11.RELEASE/reference/html/)
> [3] [Spring Data Redis 实践](https://blog.csdn.net/wlwlwlwl015/article/details/52863821)