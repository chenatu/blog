title: spring mvc处理表单
date: 2016/7/5
categories:
- coding
tags:
- java
- spring
---
在使用spring mvc时 提交表单遇到了如下问题

表单请求的headers通常有两种content type： application/x-www-form-urlencoded和multipart/form-data。前一种类似于get请求用&连接参数，通常适用于字符串，第二种就通常适用于文件和参数混合的类型。

对于第一种请求参数，Spring mvc的大多数例子都默认支持。

对于第二种Controller的函数输入参数可以采用MultipartHttpServletRequest，并在spring配置文件中添加如下配置

```
<bean id="multipartResolver"
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">

    <!-- setting maximum upload size -->
    <property name="maxUploadSize" value="100000"/>

</bean>
```

添加如下maven依赖

```
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>

<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.4</version>
</dependency>
```