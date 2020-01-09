Redis整合CacheManager
===
```xml
  <dependency>
			<groupId>org.crazycake</groupId>
			<artifactId>shiro-redis</artifactId>
			<version>3.1.0</version>
		</dependency>
```

```java
//使用自定义的cacheManager
   securityManager.setCacheManager(cacheManager());
        
    /**
     * 配置redisManager
     *
     */
    public RedisManager getRedisManager(){
        RedisManager redisManager = new RedisManager();
        redisManager.setHost("localhost");
        redisManager.setPort(6379);
        return redisManager;
    }


    /**
     * 配置具体cache实现类
     * @return
     */
    public RedisCacheManager cacheManager(){
        RedisCacheManager redisCacheManager = new RedisCacheManager();
        redisCacheManager.setRedisManager(getRedisManager());
         //设置过期时间，单位是秒，20s,
        redisCacheManager.setExpire(20);

        return redisCacheManager;
    }

```