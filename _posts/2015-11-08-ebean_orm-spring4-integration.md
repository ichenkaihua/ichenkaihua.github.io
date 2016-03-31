---
layout: post
title: "Ebean-ORM Spring4 integration"
description: "ebean-orm spring4 integration"
category: [j2ee,Spring]
tags: [Ebean-ORM,Spring]
date: 2015-11-08 19:53:03
permalink: /:year/:month/:day/:title/
---


`Ebean ORM`是一个使用纯Java实现的开源ORM框架。 Bean使用JPA注释对实体进行映射。Ebean力求让使用最简单的API帮助开发者从数据库获取有用的数据信息。`Ebean ORM`是轻量级框架，他支持源生sql、分页、大数据查询、批量插入、数据加密、json实用功能。`Ebean ORM`还支持与spring等框架集成，`Ebean orm`与spring集成后，ebean事物交给spring全局管理，省去了不少麻烦。但是官方文档对这部分一笔带过，我初次看文档时一头雾水，就是官方demo也过时有点错误，因此我选择了目前比较新版的`Ebean ORM`与`spring`集成。<!-- more -->

### 开发环境
* JDK版本:Oracle JDK8
* IDE: IDEA
* 构建工具：gradle


### 添加依赖
本例中使用`ebean-orm:6.10.3`和`spring:4.1.7.RELEASE`版本
**build.gradle**文件中配置依赖

```groovy
apply plugin: 'war'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.avaje.ebeanorm:avaje-ebeanorm-spring:4.5.3'
    compile 'com.h2database:h2:1.4.189'
    compile "org.springframework:spring-orm:4.1.7.RELEASE"
    compile 'org.springframework:spring-aop:4.1.7.RELEASE'
    testCompile 'org.springframework:spring-test:4.1.7.RELEASE'
    compile 'org.springframework:spring-webmvc:4.1.7.RELEASE'
    compile 'org.aspectj:aspectjweaver:1.8.6'
    compile 'com.fasterxml.jackson.core:jackson-databind:2.6.1'
}

```
>**注意**:`org.avaje.ebeanorm:avaje-ebeanorm-spring:4.5.3`依赖`ebean ORM`，`ebeanorm-spring`是ebean提供的spring集成工具。

### Spring集成Ebean ORM
**spring**集成**ebean orm**,其实就是把`ebeaon orm`的事物交由spring管理。这里使用xml配置的方式配置spring，`spring-config.xml`为spring配置文件。
在**spring-config.xml** 文件中，内容如下

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
>为了配置更简单查看，我把ebean配置部分独立到`spring-ebean.xml`文件

**spring-ebean.xml**文件内容如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xmlns:tx="http://www.springframework.org/schema/tx"
	   xmlns:aop="http://www.springframework.org/schema/aop"
	   xsi:schemaLocation="
        http://www.springframework.org/schema/beans     
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.1.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-4.1.xsd
        ">

	<bean id="dataSource" class="org.springframework.jdbc.datasource.LazyConnectionDataSourceProxy">
		<constructor-arg>
			<bean class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
				<property name="driverClass" value="org.h2.Driver" />
				<property name="url"
						  value="jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1" />
			</bean>
		</constructor-arg>
	</bean>

	<!--  Transaction Manager -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>

	<bean id="serverConfig" class="com.avaje.ebean.config.ServerConfig">
		<property name="externalTransactionManager">
			<bean class="com.avaje.ebean.springsupport.txn.SpringAwareJdbcTransactionManager"/>
		</property>
		<property name="defaultServer" value="true"/>

		<property name="namingConvention">
			<bean class="com.avaje.ebean.config.UnderscoreNamingConvention"/>
		</property>
		<property name="name" value="ebeanServer"/>

		<property name="packages">
			<list>
				<value>com.chenkaihua.springebean.entity</value>
			</list>
		</property>

		<property name="dataSource" ref="dataSource"/>
		<!--<property name="disableClasspathSearch" value="true"/>-->
		<!--是否生成sql文件-->
		<property name="ddlGenerate" value="false"/>
		<!--时候启动时读取sql文件，并执行-->
		<property name="ddlRun" value="false"/>
	</bean>

	<!-- Ebean server -->
	<bean id="ebeanServer" class="com.avaje.ebean.springsupport.factory.EbeanServerFactoryBean">
		<property name="serverConfig" ref="serverConfig"/>
	</bean>

	<aop:aspectj-autoproxy  />

	<aop:config>
		<aop:pointcut id="appService"
					  expression="execution(* com.chenkaihua.springebean..*Service*.*(..))" />
		<aop:advisor advice-ref="txAdvice" pointcut-ref="appService" />
	</aop:config>

	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="select*" read-only="true" />
			<tx:method name="find*" read-only="true" />
			<tx:method name="get*" read-only="true" />
			<tx:method name="*" />
			<tx:method name="sava*"  />
			<tx:method name="update*" />
			<tx:method name="delete*" />
		</tx:attributes>
	</tx:advice>


</beans>
```
> **注意** :配置使用的`dataSource`类型为`org.springframework.jdbc.datasource.LazyConnectionDataSourceProxy`,这是一个代理类，spring与ebean-orm集成时必须使用这个类，不然事物不起作用。`ebeanServer`是ebean-ORM的核心类，也是查询的核心接口。

这样配置之后，就可以在service里注入`EbeanServer`进行数据库操作了。

### 怎样使用

ebean项目部署前，必须要`enhance`，具体操作请参考我的教程：[ebean-orm enhance with gradle]({% post_url 2015-11-08-ebean_orm-enhance-with-gradle %})

```java
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
如果调用`saveOnThrowException()`方法，抛出异常，事物回滚，说明spring事物起作用了。

### 参考资料
> * ebean官网文档: [http://ebean-orm.github.io][ebean-doc]
* ebean-github: [ https://github.com/ebean-orm/avaje-ebeanorm][ebean-github]


[ebean-doc]: http://ebean-orm.github.io/
[ebean-github]: https://github.com/ebean-orm/avaje-ebeanorm