title: 像@Transactional一样利用注解自定义aop切片
date: 2016/8/2
categories:
- coding
tags:
- java
- spring
---
在spring中，利用@Transactional注解可以很轻松的利用aop技术进行事物管理。在实际项目中，直接利用自定义注解实现切片可以大大的提高我们的编码效率以及代码的简洁性。

实现以上的目标，主要涉及两方面工作。
1. 自定义注解
2. 将注解声明为切片


## 自定义注解

介绍注解自定义的文章比较多，这里简要介绍一下以下面的代码为例。该代码要实现一个分布式锁的代码。首先利用`@interface`来声明该类为接口类，用`@Target`来声明该注解的作用范围。该例子中`ElementType.METHOD, ElementType.TYPE`表明该注解作用范围是方法和类。`@Retention`注明该注解的作用周期。`RetentionPolicy.RUNTIME`表明该注解在运行时也是有效的。

```
package com.whaty.lock.annotation;

import org.springframework.stereotype.Component;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Component
@Target(value = {ElementType.METHOD, ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface Lock {

    public LockImpl lockImpl() default LockImpl.MYSQL;

    public enum LockImpl {
        MYSQL, ZOOKEEPER
    }
}
```

## 将注解声明为切片
下面的代码是实现注解切片逻辑的代码。其中@Aspect用来声明切片的实现。在这个代码里面，最关键的一步是
`@Around(value = "@annotation(com.whaty.lock.annotation.Lock)")`
这个声明与普通的注解式声明切片类似，只是其中`@annotation`表明该切片作用范围为声明的注解作用范围。

```
package com.whaty.lock.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class LockAspect {
    @Around(value = "@annotation(com.whaty.lock.annotation.Lock)")
    void execute(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        // 尝试获取锁
        proceedingJoinPoint.proceed();
        // 释放锁
    }
}
```