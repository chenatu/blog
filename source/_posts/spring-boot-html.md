title: spring boot直接返回静态html
date: 2016/9/26
categories:
- coding
tags:
- java
- spring
---
通常spring boot的一般教程的例子都是通过模板来返回页面，比如thymeleaf或者freemarker，但是直接返回html的例子比较少。本文参考文章[SpringBoot : How to display static html file in Spring boot MVC application](http://www.ekiras.com/2016/06/how-to-display-static-html-in-springboot-mvc.html)。说明如何让spring boot直接返回html。

一般来说`resources/static`或者`resources/public`文件夹可以用来提供`js`,`css`,图片等文件访问。不经过配置，直接返回`html`会报404错误。提供静态html访问主要需要如下配置（懒得翻译了。。。）

- You should create a class that extends `WebMvcConfigurerAdapter`
- Your class should have `@Configuration` annotation.
- You class should not have `@EnableMvc` annotation.
- Override `addViewControllers` method and add your mapping.
- Override `configurePathMatch` method and update suffix path matching

其实，添加如下配置类就好了

````
@Configuration  
public class MvcConfigurer extends WebMvcConfigurerAdapter {  
  
    @Override  
    public void addViewControllers(ViewControllerRegistry registry) {  
        registry.addViewController("/error").setViewName("error.html");  
        registry.setOrder(Ordered.HIGHEST_PRECEDENCE);  
    }
    
    @Override  
    public void configurePathMatch(PathMatchConfigurer configurer) {  
        super.configurePathMatch(configurer);  
        configurer.setUseSuffixPatternMatch(false);  
    }
}
````