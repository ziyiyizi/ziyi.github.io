---
title: springboot redis
author: ziyi
tags:
  - spring boot
  - redis
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
  - redis
comments: true
---

# springboot redis

## properties

```
spring.redis.database=0
spring.redis.host=xxx
spring.redis.port=6379
spring.redis.password=
```

## pom

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

## use
```
@Resource
private StringRedisTemplate stringRedisTemplate;

stringRedisTemplate.opsForValue().set(AccountConst.TRANSACTION_ID + transactionId, TransactionalMessageStatus.commit.getMessage());
String val = stringRedisTemplate.opsForValue().get(AccountConst.TRANSACTION_ID + transactionalId);
```

## increment

`stringRedisTemplate.opsForValue().increment(KEY, INC_NUM)`

- 原子性加值
- key不存在时会初始化为0

## redisson(集群配置)
```
# redisson
spring.redis.password =
spring.redis.database = 1
spring.redis.cluster.max-redirects = 3
spring.redis.sentinel.master = master-1
spring.redis.sentinel.nodes = #ip:port
```
```
@Configuration
public class RedissonConfig {

    @Value("${isProd:false}")
    private boolean isProd;

    @Value("${spring.redis.sentinel.nodes}")
    private String redisUrl;

    @Value("${spring.redis.password}")
    private String password;

    @Value("${spring.redis.database}")
    private int database;

    @Value("${spring.redis.sentinel.master}")
    private String masterName;

    @Bean("redisson")
    public  Redisson getRedisson() {
        Config config = new Config();
        String prefix = "redis://";
        String[] nodes = redisUrl.split(",");
        List<String> nodesList = new ArrayList<>(nodes.length);
        Arrays.stream(nodes).forEach(node -> nodesList.add(node.startsWith(prefix) ? node : prefix + node));
        if (isProd) {
            nodes = nodesList.toArray(new String[0]);
            config.useSentinelServers().setMasterName(masterName).addSentinelAddress(nodes)
                    .setPassword(password)
                    .setDatabase(database);
        } else {
            config.useSingleServer().setDatabase(database).setAddress(nodesList.get(0).toString()).setPassword(password);
        }
        return (Redisson)Redisson.create(config);
    }
}
```
```
@Resource(name = "redisson")
private Redisson redisson;

RLock rLock = redisson.getLock(LOCK_NAME);
boolean isLock = rLock.tryLock(1, 60 * 60, TimeUnit.SECONDS);
if (!isLock) {
    log.error("定时任务锁已被其他机器相同服务占有，任务终止。");
    return;
}
```