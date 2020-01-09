自定义Filter
===
```java
public class CustomRolesOrAuthorizationFilter extends AuthorizationFilter {  
  
    @Override  
    protected boolean isAccessAllowed(ServletRequest req, ServletResponse resp, Object mappedValue) throws Exception {  
        
        Subject subject = getSubject(req, resp);

        //获取当前访问路径所需要的角色集合  
        String[] rolesArray = (String[]) mappedValue;  
  
  		//没有角色限制，有权限访问  
        if (rolesArray == null || rolesArray.length == 0) { 
            return true;  
        }  

        //当前subject是rolesArray中的任何一个，则有权限访问  
        for (int i = 0; i < rolesArray.length; i++) {  
            if (subject.hasRole(rolesArray[i])) { 
                return true;  
            }  
        }  
  
        return false;  
    }  
}
```
```java
//FactoryBean配置
Map<String, Filter> filtersMap = new LinkedHashMap<>();
filtersMap.put("roleOrFilter",new CustomRolesOrAuthorizationFilter());
filterChainDefinitionMap.put("/admin/**","roleOrFilter[admin,root]");
shiroFilterFactoryBean.setFilters(filtersMap);
```