title: Spring boot中如何定义过滤器Filter
date: 2016/7/8
categories:
- coding
tags:
- java
- spring
---
最近刚刚接手使用spring boot，真是一个开发很顺手的工具。在这里总结一下自己发现的基于`@Configuration`的注解定义

```
package example.hello;

import org.springframework.boot.context.embedded.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.ArrayList;
import java.util.List;

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean greetingFilterRegistrationBean() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setName("greeting");
        GreetingFilter greetingFilter = new GreetingFilter();
        registrationBean.setFilter(greetingFilter);
        registrationBean.setOrder(1);
        List<String> urlList = new ArrayList<String>();
        urlList.add("/abc");
        registrationBean.setUrlPatterns(urlList);
        return registrationBean;
    }

    @Bean
    public FilterRegistrationBean helloFilterRegistrationBean() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setName("hello");
        HelloFilter helloFilter = new HelloFilter();
        registrationBean.setFilter(helloFilter);
        registrationBean.setOrder(2);
        return registrationBean;
    }

    /*
    @Bean
    @Order(1)
    Filter greetingFilter() {
        return new GreetingFilter();
    }

    @Bean
    @Order(2)
    public Filter helloFilter() {
        return new HelloFilter();
    }*/

}

```

其中`GreetingFilter`和`HelloFiter`是定义的简单的打印字符串过滤器。在`@Configuration`中，声明注解`@Bean`相当于在Spring老版本中在配置文件中声明一个Bean。

在这里展示了两种过滤器声明方式，第一种利用`FilterRegistrationBean`可以详细地更好地详细的定义过滤器。第二种注释掉的，声明方式更简单，代码更加简洁。

在这里也咨询大家一个问题，用第二种方式如何声明UrlPattern呢，貌似没有相关的注解