整合springboot
===
## 添加依赖
```xml
<dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.4.0</version>
</dependency>
```
## QuickStart
```java
/**
 * 单元测试执行顺序
 * @BeforeClass-> @Before-> @Test->@After->@AfterClass
 *
 */
public class QuickStart1 {
   private SimpleAccountRealm  accountRealm=  new SimpleAccountRealm();

   private DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();

    @Before
    public void init(){
        //初始化数据源
        accountRealm.addAccount("king","123");
        accountRealm.addAccount("hn","123");
        //构建环境
        defaultSecurityManager.setRealm(accountRealm);
    }

    @Test
    public void testAuth(){
        SecurityUtils.setSecurityManager(defaultSecurityManager);

        Subject subject = SecurityUtils.getSubject();
        //用户输入
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("king","123");

        subject.login(usernamePasswordToken);
        System.out.println(subject.isAuthenticated());
    }

}
```
```java
/**
 * 单元测试执行顺序
 * @BeforeClass-> @Before-> @Test->@After->@AfterClass
 *
 */
public class QuickStart2 {
   private SimpleAccountRealm  accountRealm=  new SimpleAccountRealm();

   private DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();

    @Before
    public void init(){
        //初始化数据源
        accountRealm.addAccount("king","123","root","admin");
        accountRealm.addAccount("hn","123", "user");
        //构建环境
        defaultSecurityManager.setRealm(accountRealm);
    }

    @Test
    public void testAuth(){
        SecurityUtils.setSecurityManager(defaultSecurityManager);

        Subject subject = SecurityUtils.getSubject();
        //用户输入
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("king","123");

        subject.login(usernamePasswordToken);
        System.out.println(subject.isAuthenticated());
        System.out.println(subject.hasRole("root"));
        System.out.println(subject.getPrincipal());
        subject.logout();
        System.out.println(subject.hasRole("root"));
    }

}
```