---
layout: post
title: 测试页面
tagline: test code
tags : [test]
---
{% include JB/setup %}
# 测试页面2
这是个测试页面，测试代码如下
java代码测试

```java
package com.chenkaihua.springebean.service;

import com.avaje.ebean.EbeanServer;
import com.chenkaihua.springebean.entity.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * Created by chenkaihua on 15-10-29.
 */
@Service
public class UserService {


    @Autowired
    EbeanServer ebeanServer;

    public void save(User user){
        ebeanServer.save(user);
    }

    public void saveOnThrowException(User user){
        ebeanServer.save(user);
        throw new IllegalArgumentException("非法参数！！");
    }


    public List<User> users(){
        return ebeanServer.find(User.class).findList();
    }

}
```


xml代码测试:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans     
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.2.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.1.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-4.1.xsd
        ">

    <!-- 扫描注解，除去web层注解，web层注解在mvc配置中扫描 -->
    <context:component-scan
        base-package="com.chenkaihua.springebean.service">
        <context:exclude-filter type="annotation"
            expression="org.springframework.stereotype.Controller" />
        <context:exclude-filter type="annotation"
            expression="org.springframework.web.bind.annotation.RestController" />
    </context:component-scan>

    <!-- 开启AOP监听 只对当前配置文件有效 -->
    <aop:aspectj-autoproxy expose-proxy="true" proxy-target-class="true" />


    <tx:annotation-driven transaction-manager="transactionManager"  />

    <import resource="spring-ebean.xml"/>
</beans>
```

gradle代码测试

```groovy

# 启动项目
./gradlew appStart
# 插入数据
curl http://localhost:8080/users/save
# 插入数据（抛出exception，事物回滚）
curl http://localhost:8080/users/saveE
# 浏览所以数据
curl http://localhost:8080/users
# 关闭server
./gradlew appStop
```
