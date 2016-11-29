title: 单数据源访问多数据库的路由开发
date: 2016/7/5
categories:
- coding
tags:
- java
----
在某些可以配置多站点的开发框架中，如果每个站点单独配置了单独的数据库。那么利用单一数据源根据不同的站点切换不同的数据库比较方便。

在这里展示了spring框架下的解决方案。利用了spring的`org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource`

站点路由的datasource `SiteRoutingDataSource`

```

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

public class SiteRoutingDataSource extends AbstractRoutingDataSource {

  @Override
  protected Object determineCurrentLookupKey() {
    return SiteContextHolder.getSiteCode();
  }
}
```

用来判定当前站点的工具类`SiteContextHolder`

```
import org.springframework.util.Assert;

public class SiteContextHolder {

  private static final ThreadLocal<String> contextHolder =
    new ThreadLocal<String>();

  public static void setSiteCode(String siteCode) {
    Assert.notNull(siteCode, "siteCode cannot be null");
    contextHolder.set(siteCode);
  }

  public static String getSiteCode() {
    return (String) contextHolder.get();
  }

  public static void clearSiteCode() {
    contextHolder.remove();
  }
}
```

spring的xml配置：
```
    <bean id="hgc"
          class="com.mchange.v2.c3p0.ComboPooledDataSource"
          destroy-method="close">
        <property name="driverClass">
            <value>${hgc.datasource.driverClassName}</value>
        </property>
        <property name="jdbcUrl">
            <value>${hgc.datasource.url}</value>
        </property>
        <property name="user">
            <value>${hgc.datasource.username}</value>
        </property>
        <property name="password">
            <value>${hgc.datasource.password}</value>
        </property>
    </bean>
    <bean id="ahpu"
          class="com.mchange.v2.c3p0.ComboPooledDataSource"
          destroy-method="close">
        <property name="driverClass">
            <value>${ahpu.datasource.driverClassName}</value>
        </property>
        <property name="jdbcUrl">
            <value>${ahpu.datasource.url}</value>
        </property>
        <property name="user">
            <value>${ahpu.datasource.username}</value>
        </property>
        <property name="password">
            <value>${ahpu.datasource.password}</value>
    </bean>
        <bean id="dataSource" class="SiteRoutingDataSource">
        <property name="targetDataSources">
            <map key-type="java.lang.String">
                <entry key="hgc" value-ref="hgc"/>
                <entry key="ahpu" value-ref="ahpu"/>
            </map>
        </property>
        <property name="defaultTargetDataSource" ref="ahpu"/>
    </bean>
```

在使用过程中 通过 `SiteContextHolder.setSiteCode(CODE);`来进行数据源选择。在切换数据库之前，需要先`SiteContextHolder.clearSiteCode();`再进行切换