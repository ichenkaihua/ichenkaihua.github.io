---
layout: post
title: "spring4 test 测试框架使用"
description: "spring test framework basic usage"
category: spring
tags: [spring test,junit,springMVC Test]
date: 2016-01-02 00:51:01
permalink: /:year/:month/:day/:title/
---
spring作为最负盛名的java框架，自然有配套的测试框架，这就是`Spring Test`框架。spring测试框架整合`junit`,`jmock`等单元测试框架,为开发人员节省了大量时间。Spring测试框架还包括springMVC的 web测试，引入springMVC测试框架后，应用无需在j2ee容器中启动即可断言调试，并且支持事物回滚。<!-- more -->

## 添加依赖
在spring项目中，首先需要加入`Spring Test`框架依赖。下面是gradle配置依赖的写法。

```groovy
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    testCompile 'org.springframework:spring-test:4.1.7.RELEASE'
}
```

## 集成单元测试

为了重用代码，通常写一个基类来配置测试环境，代码如下:

```java
package com.chenkaihua.test.unit;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.transaction.TransactionConfiguration;
import org.springframework.transaction.annotation.Transactional;

/**
 * Created by chenkaihua on 15-11-27.
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring-config.xml")
@TransactionConfiguration(transactionManager = "transactionManager",defaultRollback = true)
@Transactional
public class BaseUnitTest  {

    @Test
    public void test(){

    }

}
```
下面解释下主要注解:

* `@RunWith`: 用于指定junit运行环境，是junit提供给其他框架测试环境接口扩展，spring
为了便于使用spring的依赖注入，spring提供了`org.springframework.test.context.junit4.SpringJUnit4ClassRunner`作为junit测试环境。
* `@ContextConfiguration`:用于指定spring配置环境。
* `@TransactionConfiguration`:用于配置事物。
* `@Transactional`:表示所有类下所有方法都使用事物。

### 编写单元测试
个人认为，单元测试到`service`级别最好，我通常是一个`service`配套一个单元测试,下面是一个简单的测试，使用了spring注入

```java
package com.chenkaihua.test.unit;

import com.dreamliner.mhml.entity.User;
import com.dreamliner.mhml.mapper.ImageResourceMapper;
import com.dreamliner.mhml.service.UserService;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;

import static org.junit.Assert.*;

/**
 * Created by chenkaihua on 15-11-27.
 */
public class UserServiceTest extends BaseUnitTest {


    @Autowired
    UserService userService;

    @Autowired
    ImageResourceMapper imageResourceMapper;



    @Test
    public void testLogin() throws  Exception{
        User selectUser = new User();
        selectUser.setPhone("1111111111");
        selectUser.setPassword("pass");
        User user = userService.selectWithAvatar(selectUser);
        assertNotNull(user);
        assertNotNull(user.getAvatar());
    }

}
```
可以看到，`UserServiceTest`继承单元测试基类`BaseUnitTest`，由于`BaseUnitTest`配置了测试环境，因此这里可以直接使用spring注入注解，是不是方便了很多！

## 集成MVC测试
首先，也是编写一个基类，用于配置测试环境

```java
package com.chenkaihua.test.web;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.ContextHierarchy;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.transaction.TransactionConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.context.WebApplicationContext;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.webAppContextSetup;

/**
 * Created by chenkaihua on 15-10-29.
 */

@WebAppConfiguration
@Transactional(value = "transactionManager")
@TransactionConfiguration(defaultRollback = true,transactionManager = "transactionManager")
@RunWith(SpringJUnit4ClassRunner.class)
@ContextHierarchy({
        @ContextConfiguration("classpath:spring-config.xml"),
        @ContextConfiguration("classpath:spring-mvc-config.xml")
})
public class WebBaseTest {

    @Autowired
    protected WebApplicationContext wac;

    protected MockMvc mockMvc;


    @Before
    public void setup() {
        this.mockMvc = webAppContextSetup(this.wac).build();
    }

    @Test
    public void test() throws Exception {


    }

}
```
有几个注意的地方

* `@WebAppConfiguration`:指定是mvc测试环境
*  `ContextHierarchy`:指定spring配置文件层次结构，即`spring-mvc-config.xml`上下文继承`pring-config.xml`,也就是把`springMVC`环境配置文件加入进来。
*  `WebApplicationContext`:模拟的webapp context。
* `MockMvc`:一个模拟的MVC环境。

### 编写mvc测试用例
在web测试中，主要是基于`Controller`写的web接口测试。

```java
package com.chenkaihua.test.web;

import com.dreamliner.mhml.service.UserService;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mock.web.MockMultipartFile;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.fileUpload;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

/**
 * Created by chenkaihua on 15-12-12.
 */
public class UserControllerTest extends WebBaseTest {


    @Autowired
    UserService userService;

    @Test
    public void testRegister() throws Exception {

        mockMvc.perform(post("/users").param("phone", "11111111111").param("password", "passssss")).andDo(print())
                .andExpect(status().isOk()).andReturn();
    }
}

```
这是个测试注册的方法，被测试`UserController`代码如下:

```java
 @RequestMapping(method = RequestMethod.POST)
    public ResponseEntity register(@RequestParam String phone, @RequestParam String password) {
        boolean register = userService.isRegister(phone);
        if (register) {
            return ResponseEntity.status(Status.STATUS_1).body("user  has register");
        }
        User user = new User();
        user.setPhone(phone);
        user.setName("user" + phone.substring(0,2)+"******"+phone.substring(phone.length()-3,phone.length()-1));
        user.setPassword(password);
        userService.insert(user);
        return ResponseEntity.ok(user);
    }
```

*  `mockMvc.perform()`: 用于构造一个测试请求,即模拟一个`request`。可以指定HTTP方法(`POST`,`GET`,`DELETE`,`PUT`等)，路径，参数等。
* `andDo(print())`: 请求完后，打印结果。运行测试时，会在控制台打印请求详情，已经被调用的`Controller`的具体信息。
* `andExpect(status().isOk())`:请求完后的断言，判断返回的状态码是否为`200`

### 文件上传测试
web开发中，文件上传是基本功能。spring mvc测试框架中可以很方便的测试文件上传

```java
   @Test
    public void testUploadAvatar() throws Exception {

        MockMultipartFile mockMultipartFile = new MockMultipartFile("avatarFile", "testsssssss".getBytes());
        mockMvc.perform(fileUpload("/users/1/avatar").
                file(mockMultipartFile).param("type", "img"))

                .andDo(print()).andExpect(status().isOk()).andReturn();
    }
```

* `MockMultipartFile`:模拟一个文件出出来，可以传`byte[]`构造
* `fileUpload()`:以post方式提交`multipart request`

`mvc`测试非常强大，还可以设定`session`等。总之，`request`的所有属性都可以模拟。

## 参考资料
>* [integration-testing--Spring官方文档][2]
> * [Spring MVC测试框架详解——服务端测试 - 开涛的博客][1]


[1]: http://jinnianshilongnian.iteye.com/blog/2004660
[2]: http://docs.spring.io/spring/docs/current/spring-framework-reference/html/integration-testing.html