ShiroSession
===
* 什么是会话session
    * 用户和程序直接的链接，程序可以根据session识别到哪个用户，和javaweb中的session类似
* 什么是会话管理器SessionManager
    * 会话管理器管理所有subject的所有操作，是shiro的核心组件
    * 核心方法：
    ```java
    //开启一个session
    Session start(SessionContext context);
    //指定Key获取session
    Session getSession(SessionKey key)
    ```
* SessionDao 会话存储/持久化
    * SessionDAO AbstractSessionDAO CachingSessionDAO EnterpriseCacheSessionDAO MemorySessionDAO
    * 核心方法
    ```java
    //创建
    Serializable create(Session session);
    //获取
    Session readSession(Serializable sessionId) throws UnknownSessionException;
    //更新
    void update(Session session) 
    //删除，会话过期时会调用
    void delete(Session session);
    //获取活跃的session
    Collection<Session> getActiveSessions();
    ```
## RememberMe
* Shiro 提供了记住我（RememberMe）的功能，比如访问如淘宝等一些网站时，关闭了浏览器，下次再打开时还是能记住你是谁，下次访问时无需再登录即可访问，基本流程如下：
    * 首先在登录页面选中 RememberMe 然后登录成功；如果是浏览器登录，一般会把 RememberMe 的Cookie 写到客户端并保存下来；
    * 关闭浏览器再重新打开；会发现浏览器还是记住你的；
    * 访问一般的网页服务器端还是知道你是谁，且能正常访问；
* subject.isAuthenticated() 表示用户进行了身份验证登录的，即使用 Subject.login 进行了登录；
* subject.isRemembered()：表示用户是通过记住我登录的，此时可能并不是真正的你（如你的朋友使用你的电脑，或者你的cookie 被窃取）在访问的；
* 两者二选一，即 subject.isAuthenticated()==true，则subject.isRemembered()==false；反之一样。
### 配置
#### 步骤1：在ShiroConfiguration配置中 创建Cookie(记住我)管理器@Bean
```java
**
* cookie管理器;
* @return
*/
@Bean
public CookieRememberMeManager rememberMeManager(){
    logger.info("注入Shiro的记住我(CookieRememberMeManager)管理器-->rememberMeManager", CookieRememberMeManager.class);
    CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
    //rememberme cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度（128 256 512 位），通过以下代码可以获取
    //KeyGenerator keygen = KeyGenerator.getInstance("AES");
    //SecretKey deskey = keygen.generateKey();
    //System.out.println(Base64.encodeToString(deskey.getEncoded()));
    byte[] cipherKey = Base64.decode("wGiHplamyXlVB11UXWol8g==");
    cookieRememberMeManager.setCipherKey(cipherKey);
    cookieRememberMeManager.setCookie(rememberMeCookie());
    return cookieRememberMeManager;
}
@Bean
public SimpleCookie rememberMeCookie(){
    //这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
    SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
    //如果httyOnly设置为true，则客户端不会暴露给客户端脚本代码，使用HttpOnly cookie有助于减少某些类型的跨站点脚本攻击；
    simpleCookie.setHttpOnly(true);
    //记住我cookie生效时间,默认30天 ,单位秒：60 * 60 * 24 * 30
    simpleCookie.setMaxAge(259200);

    return simpleCookie;
}


```
#### 步骤2：将Cookie管理器注入到SecurityManager中
```java
@Bean
public SecurityManager securityManager() {
    logger.info("注入Shiro的Web过滤器-->securityManager", ShiroFilterFactoryBean.class);
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    //设置Realm，用于获取认证凭证
    securityManager.setRealm(userRealm());
    //注入缓存管理器
    securityManager.setCacheManager(ehCacheManager());
    //注入Cookie(记住我)管理器(remenberMeManager)
    securityManager.setRememberMeManager(rememberMeManager());

    return securityManager;
}
```
#### 步骤3：当访问指定路径资源时，只需要记录我就能访问，则在ShiroFilterFactoryBean添加记住我过滤器:user
```java
@Bean(name = "shiroFilter")
public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
    ...
    Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
    // 配置退出过滤器,其中的具体的退出代码Shiro已经替我们实现了
    filterChainDefinitionMap.put("/logout", "logout");
    //配置记住我过滤器或认证通过可以访问的地址(当上次登录时，记住我以后，在下次访问/或/index时，可以直接访问，不需要登陆)
    filterChainDefinitionMap.put("/index", "user");
    filterChainDefinitionMap.put("/", "user");
    //过滤链定义，从上向下顺序执行，一般将 /**放在最为下边 -->:这是一个坑呢，一不小心代码就不好使了;
    ...
    return shiroFilterFactoryBean;
}

```
#### 步骤4：目前我们采用的是创建Token的方式来登陆：UsernamePasswordToken(用户名，密码，是否记住我)，在创建Token后，加上token.setRememberMe(true),修改SecurityController; 
```java
@RequestMapping(value="/login",method=RequestMethod.POST)
public String login(HttpServletRequest request, @Valid User user,BindingResult bindingResult,RedirectAttributes redirectAttributes){
    ...
    UsernamePasswordToken token = new UsernamePasswordToken(user.getUserName(), user.getPassword());
    if(request.getParameter("rememberMe")!=null){
        token.setRememberMe(true);
    }
    //获取当前的Subject  
    Subject currentUser = SecurityUtils.getSubject();  
    ...
    }
```
#### 步骤5： 修改login.html
```html

 <br />
    <!--下面是记住我输入框-->
    <div class="checkbox">
      <label>
        <input id="rememberMe" name="rememberMe" type="checkbox" /> Remember me
      </label>
    </div>
    <br />
    <!--下面是登陆按钮,包括颜色控制-->
    <button type="submit" style="width:280px;" class="btn btn-default">登 录</button>
```