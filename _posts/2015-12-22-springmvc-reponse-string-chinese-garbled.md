---
layout: post
title: "SpringMVC @Response 返回String 中文乱码"
description: "springMVC @Reponse String Chinese garbled"
category: [j2ee,spring]
fullwidth: true
tags: [spring,springmvc,乱码]
date: 2015-12-22 18:45:27
permalink: /:year/:month/:day/:title/
---
网上有多种解决办法，发现这种方式最简便：
在springMVC的配置文件中(`springMVC-xx.xml`)，修改`<mvc:annotation-driven />`为：

```xml
	<mvc:annotation-driven>
		<mvc:message-converters register-defaults="true">
			<bean class="org.springframework.http.converter.StringHttpMessageConverter">
				<constructor-arg value="UTF-8" />
			</bean>
		</mvc:message-converters>
	</mvc:annotation-driven>
```
**乱码原因**:`org.springframework.http.converter.StringHttpMessageConverter`有个`final`修饰的` Charset DEFAULT_CHARSET=Charset.forName("ISO-8859-1")`常量。即当返回类型为`String`时，返回类型为`text/plain`，字符被设置为默认字符`ISO-8859-1`。
