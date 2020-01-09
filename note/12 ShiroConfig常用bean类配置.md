 ShiroConfig常用bean类配置
 ===
 * LifecycleBeanPostProcessor
    * 作用：管理shiro一些bean的生命周期 即bean初始化 与销毁
   ```java
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
    	return new LifecycleBeanPostProcessor();
    }
    ```
* AuthorizationAttributeSourceAdvisor
    * 作用：加入注解的使用，不加入这个AOP注解不生效(shiro的注解 例如 @RequiresGuest)
```java
    @Bean
     public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor() {
      AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
            authorizationAttributeSourceAdvisor.setSecurityManager(securityManager());
            return authorizationAttributeSourceAdvisor;
    }
```
* DefaultAdvisorAutoProxyCreator
    * 作用: 用来扫描上下文寻找所有的Advistor(通知器), 将符合条件的Advisor应用到切入点的Bean中，需要在LifecycleBeanPostProcessor创建后才可以创建
    ```java
    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    public  DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator(){
            DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator=new DefaultAdvisorAutoProxyCreator();
            defaultAdvisorAutoProxyCreator.setUsePrefix(true);
            return defaultAdvisorAutoProxyCreator;
    }
    ```

