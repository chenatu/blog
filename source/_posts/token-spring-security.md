---
title: spirng-boot中，基于既有的token验证方式，利用spring-security实现权限系统
date: 2016-12-09 16:04:18
categories:
- coding
tags:
- java
- spring
---
用过spring-security的都应该能感觉到，spring-security把authentication和authorization封装的比较死。默认的authorization是基于session的。利用session验证过的信息，保存进SecurityContext，权限系统再根据SecurityContext保存的用户权限相关信息，来进行权限管理。

但是在目前的场景中，服务器端往往要满足多端的验证方式，session的方式不容易和移动端配合的好。更多的是用一个token放在http header中进行验证。这种就需要绕开spring-security默认的authentication直接利用它的authorization。

在这里我演示一个在spring-security做方法级别拦截的方案。

这里就是基于token的spring-boot安全拦截配置

````
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
@Order(1)
public class TokenBasedSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) {
        try {
            http.addFilterBefore(... SecurityContextPersistenceFilter.class);

            http.securityContext().securityContextRepository(new SecurityContextRepository() {
                @Override
                public SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder) {
					...
                }

                @Override
                public void saveContext(SecurityContext context, HttpServletRequest request, HttpServletResponse response) {
					...
                }

                @Override
                public boolean containsContext(HttpServletRequest request) {
					...
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
````

在这里，我们要做的其实就是设置重在`SecurityContextRepository`。这个实体在spring security启动中要传递给`SecurityContextPersistenceFilter`。这个filter根据request来加载`SecurityContext`。而`SecurityContextPersistenceFilter`就是从其内部的`SecurityContextRepository`来加载`SecurityContext`的。所以我们就需要重载上面代码中的三个方法，根据request来构造`SecurityContext`。

我们再来看一下`SecurityContext`到底封装了什么。

````
public interface SecurityContext extends Serializable {

	Authentication getAuthentication();

	void setAuthentication(Authentication authentication);
}
````
Authentication而已。

public interface Authentication extends Principal, Serializable {

	Collection<? extends GrantedAuthority> getAuthorities();

	Object getCredentials();

	Object getDetails();

	Object getPrincipal();

	boolean isAuthenticated();


	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}

在这里我们还要构造一个机遇token的Authentication接口的实现。在实现中对于权限来说很有用的就是`getAuthorities`方法。我们只要给其封装最简单的`SimpleGrantedAuthority`就好了。

这样我们就可以给我们的Controller方法做拦截了~

````
@RestController
@RequestMapping(value = "test")
public class TestController {

    @PreAuthorize("hasAuthority('super_admin')")
    @RequestMapping(value = "hello", method = RequestMethod.GET)
    public String superHello(@RequestParam String domain) {
        return new String("super hello");
    }
}

````