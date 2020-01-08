ApacheShiro基础概念
===
* 直达Apache Shiro官网 http://shiro.apache.org/introduction.html
* 什么是身份认证
    * Authentication，身份证认证，一般就是登录
* 什么是授权
    * Authorization，给用户分配角色或者访问某些资源的权限
* 什么是会话管理
    * Session Management, 用户的会话管理员，多数情况下是web session
* 什么是加密
    Cryptography, 数据加解密，比如密码加解密等

## 名词解释
* Subject
    * 我们把用户或者程序称为主体（如用户，第三方服务，cron作业），主体去访问系统或者资源
* SecurityManager
    * 安全管理器，Subject的认证和授权都要在安全管理器下进行
* Authenticator
    * 认证器，主要负责Subject的认证
* Realm
    * 数据域，Shiro和安全数据的连接器，好比jdbc连接数据库； 通过realm获取认证授权相关信息
* Authorizer
    * 授权器，主要负责Subject的授权, 控制subject拥有的角色或者权限
* Cryptography
    * 加解密，Shiro的包含易于使用和理解的数据加解密方法，简化了很多复杂的api
* Cache Manager
    * 缓存管理器，比如认证或授权信息，通过缓存进行管理，提高性能
<img src="../imgs/ShiroArchitecture.png" width="900" height="600">