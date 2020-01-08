ApacheRealm
===
## Realm
* realm作用：Shiro 从 Realm 获取安全数据
* 默认自带的realm：idae查看realm继承关系，有默认实现和自定义继承的realm
* 两个概念
    * principal : 主体的标示，可以有多个，但是需要具有唯一性，常见的有用户名，手机号，邮箱等
    * credential：凭证, 一般就是密码
    * 所以一般我们说 principal + credential 就账号 + 密码
* 开发中，往往是自定义realm , 即集成 AuthorizingRealm

## 读取ini文件中的用户以及权限 
```java
public class QuickStart3 {


    @Test
    public void testAuth(){
        //创建SecurityFactory工厂
        IniSecurityManagerFactory iniSecurityManagerFactory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager instance = iniSecurityManagerFactory.getInstance();
        SecurityUtils.setSecurityManager(instance);
        Subject subject = SecurityUtils.getSubject();
        //用户输入
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("king","123");
        subject.login(usernamePasswordToken);
        System.out.println(subject.isAuthenticated());
        System.out.println(subject.hasRole("root"));
        System.out.println(subject.getPrincipal());
        System.out.println("权限"+subject.isPermitted("video:*"));
        subject.logout();
        System.out.println(subject.hasRole("root"));
    }

}

```
```ini
# 格式 name=password,role1,role2,..roleN
[users]
king = 123, admin
hn = 234,user

# 格式 role=permission1,permission2...permissionN   也可以用通配符
# 下面配置user的权限为所有video:find,video:buy，如果需要配置video全部操作crud 则 user = video:*
[roles]
user = video:find,video:buy
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = *
```

## 从数据库中读取
### 数据库表结构
```sql
DROP TABLE IF EXISTS `roles_permissions`;
 
CREATE TABLE `roles_permissions` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_name` varchar(100) DEFAULT NULL,
  `permission` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_roles_permissions` (`role_name`,`permission`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

 
DROP TABLE IF EXISTS `user_roles`;
 
CREATE TABLE `user_roles` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(100) DEFAULT NULL,
  `role_name` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_user_roles` (`username`,`role_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


DROP TABLE IF EXISTS `users`;
 
CREATE TABLE `users` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(100) DEFAULT NULL,
  `password` varchar(100) DEFAULT NULL,
  `password_salt` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_users_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 方式一，配置ini数据库信息
```ini
#注意 文件格式必须为ini，编码为ANSI
#声明Realm，指定realm类型
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
#配置数据源
#dataSource=com.mchange.v2.c3p0.ComboPooledDataSource
dataSource=com.alibaba.druid.pool.DruidDataSource
# mysql-connector-java 5 用的驱动url是com.mysql.jdbc.Driver，mysql-connector-java6以后用的是com.mysql.cj.jdbc.Driver
dataSource.driverClassName=com.mysql.cj.jdbc.Driver
#避免安全警告
dataSource.url=jdbc:mysql://132.232.38.94:3306/shiro_demo?useSSL=false&serverTimezone=GMT%2B8&generateSimpleParameterMetadata=true
dataSource.username=root
dataSource.password=*****
#指定数据源
jdbcRealm.dataSource=$dataSource
#开启查找权限, 默认是false，不会去查找角色对应的权限
jdbcRealm.permissionsLookupEnabled=true
#指定SecurityManager的Realms实现，设置realms，可以有多个，用逗号隔开
securityManager.realms=$jdbcRealm

```
```java
@Test
    public void testAuth(){
        //创建SecurityFactory工厂
        IniSecurityManagerFactory iniSecurityManagerFactory = new IniSecurityManagerFactory("classpath:jdbcrealm.ini");
        SecurityManager instance = iniSecurityManagerFactory.getInstance();
        SecurityUtils.setSecurityManager(instance);
        Subject subject = SecurityUtils.getSubject();
        //用户输入
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("king","456");
        subject.login(usernamePasswordToken);
        System.out.println(subject.isAuthenticated());
        System.out.println(subject.hasRole("root"));
        System.out.println(subject.getPrincipal());
        System.out.println("权限"+subject.isPermitted("video:*"));
        subject.logout();
        System.out.println(subject.hasRole("root"));
    }

```
### 方式二
```java
@Test
    public void test2(){
        DefaultSecurityManager securityManager = new DefaultSecurityManager();

        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
        ds.setUrl("jdbc:mysql://132.232.38.94:3306/shiro_demo?useSSL=false&serverTimezone=GMT%2B8&generateSimpleParameterMetadata=true");
        ds.setUsername("root");
        ds.setPassword("******");

        JdbcRealm realm = new JdbcRealm();
        realm.setPermissionsLookupEnabled(true);
        realm.setDataSource(ds);
        securityManager.setRealm(realm);

        SecurityUtils.setSecurityManager(securityManager);
        Subject subject = SecurityUtils.getSubject();
        //用户输入
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("king","456");
        subject.login(usernamePasswordToken);
        System.out.println(subject.isAuthenticated());
        System.out.println(subject.hasRole("root"));
        System.out.println(subject.getPrincipal());
        System.out.println("权限"+subject.isPermitted("video:*"));
        subject.logout();
        System.out.println(subject.hasRole("root"));
    }
```

## 自定义Realm
```java
public class CustomRealm extends AuthorizingRealm {
    private final Map<String,String> userInfoMap= new HashMap<>();

    {
        userInfoMap.put("king","123");
        userInfoMap.put("tom","123");

    }
    //角色和权限
    private final Map<String,Set<String>> permissionMap= new HashMap<>();

    {
        Set<String> set1 = new HashSet<>();
        Set<String> set2 = new HashSet<>();
        set1.add("video:find");
        set1.add("video:buy");
        set2.add("video:add");
        set2.add("video:delete");
        permissionMap.put("jack", set1);
        permissionMap.put("king", set2);

    }
    //用户和角色
    private final Map<String,Set<String>> roleMap= new HashMap<>();

    {
        Set<String> set1 = new HashSet<>();
        Set<String> set2 = new HashSet<>();
        set1.add("role1");
        set1.add("role2");
        set2.add("admin");
        roleMap.put("jack", set1);
        roleMap.put("king", set2);

    }
    //权限校验
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String name = (String) principals.getPrimaryPrincipal();
        Set<String> roles = getRolesByNameFromDB(name);
        //正确操作应该是通过角色查找权限
        Set<String> permissions = getPermissionsByNameFromDB(name);
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.setRoles(roles);
        simpleAuthorizationInfo.setStringPermissions(permissions);
        return simpleAuthorizationInfo;
    }

    /**
     * 模拟
     * @param name
     * @return
     */
    private Set<String> getRolesByNameFromDB(String name) {
        return roleMap.get(name);

    }

    /**
     * 模拟
     * @param name
     * @return
     */
    private Set<String> getPermissionsByNameFromDB(String name) {
       return  permissionMap.get(name);
    }

    //登录
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String name  = (String) token.getPrincipal();
        //模拟
        String passsword = getPwdByNameFromDB(name);
        if(passsword==null || "".equals(passsword)){
            return null;
        }
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(name,passsword,this.getName());

        return simpleAuthenticationInfo;
    }

    private String getPwdByNameFromDB(String name) {
        return userInfoMap.get(name);
    }
}
```

