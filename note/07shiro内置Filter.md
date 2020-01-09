shiro内置Filter
===
## DefaultFilter
* authc：org.apache.shiro.web.filter.authc.FormAuthenticationFilter
    * 需要认证登录才能访问
* user：org.apache.shiro.web.filter.authc.UserFilter
    * 用户拦截器，表示必须存在用户。login或rememberme
* anon：org.apache.shiro.web.filter.authc.AnonymousFilter
    * 匿名拦截器，不需要登录即可访问的资源，匿名用户或游客，一般用于过滤静态资源。
* roles：org.apache.shiro.web.filter.authz.RolesAuthorizationFilter
    * 角色授权拦截器，验证用户是或否拥有角色。
    * 参数可写多个，表示某些角色才能通过，多个参数时写 roles["admin,user"]，当有多个参数时必须每个参数都通过才算通过
* perms：org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter
    * 权限授权拦截器，验证用户是否拥有权限
    * 参数可写多个，表示需要某些权限才能通过，多个参数时写 perms["user, admin"]，当有多个参数时必须每个参数都通过才算可以
* authcBasic：org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter
    * httpBasic 身份验证拦截器。
* logout：org.apache.shiro.web.filter.authc.LogoutFilter
    * 退出拦截器，执行后会直接跳转到shiroFilterFactoryBean.setLoginUrl(); 设置的 url
* port：org.apache.shiro.web.filter.authz.PortFilter
    * 端口拦截器, 可通过的端口。
* ssl：org.apache.shiro.web.filter.authz.SslFilter
    * ssl拦截器，只有请求协议是https才能通过。

## shiro配置路径
* 路径通配符支持 `?`、`*`、`**`，注意通配符匹配不 包括目录分隔符“/”
* `*`可以匹配所有，不加`*`可以进行前缀匹配，但多个冒号就需要多个 `*` 来匹配

```
URL权限采取第一次匹配优先的方式
? : 匹配一个字符，如 /user? , 匹配 /user3，但不匹配/user/;
* : 匹配零个或多个字符串，如 /add* ,匹配 /addtest，但不匹配 /user/1
** : 匹配路径中的零个或多个路径，如 /user/** 将匹 配 /user/xxx 或 /user/xxx/yyy

例子
/user/**=filter1
/user/add=filter2
请求 /user/add  命中的是filter1拦截器
先命中就取哪个
```

## shiro权限控制注解和编程方式
* @RequiresRoles(value={"admin", "editor"}, logical= Logical.AND)
    * 需要角色 admin 和 editor两个角色 AND表示两个同时成立 ,logical= Logical.OR成立一个即可
* @RequiresPermissions (value={"user:add", "user:del"}, logical= Logical.OR)
    * 需要权限 user:add 或 user:del权限其中一个，OR是或的意思。
* @RequiresAuthentication
    * 已经授过权，调用Subject.isAuthenticated()返回true
* @RequiresUser
    * 身份验证或者通过记住我登录的

## 编程方式
```java
Subject subject = SecurityUtils.getSubject(); 
//基于角色判断
if(subject.hasRole(“admin”)) {
	//有角色，有权限
} else {
	//无角色，无权限
	
}
//或者权限判断
if(subject.isPermitted("/user/add")){
    //有权限
}else{
    //无权限
}
```