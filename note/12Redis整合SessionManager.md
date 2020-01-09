Redis整合SessionManager
===
* 为啥session也要持久化？
    * 重启应用，用户无感知，可以继续以原先的状态继续访问

```java
  //配置session持久化
   customSessionManager.setSessionDAO(redisSessionDAO());

 	
 	/**
     * 自定义session持久化
     * @return
     */
    public RedisSessionDAO redisSessionDAO(){
        RedisSessionDAO redisSessionDAO = new RedisSessionDAO();
        redisSessionDAO.setRedisManager(getRedisManager());
        return redisSessionDAO;
    }
```