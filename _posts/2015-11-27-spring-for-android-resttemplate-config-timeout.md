---
layout: post
title: Spring for android RestTemplate 超时设置
description: "spring for android resttemplate config timeout"
category: android
tags: [Spring,android annotations]
date: 2015-11-27 00:09:40
permalink: /:year/:month/:day/:title/
---

**spring for android**是一个android平台下的网络框架，由大名鼎鼎的spring项目组开发。**spring for android**对于处理rest网络请求特别方便，这是我使用**spring for android**的主要原因。使用过程中，可能发现spring for android这套框架不好设置超时，有时甚至设置了也无效。如果使用了aa注解框架，设置超时更麻烦点。下面从源码角度解读这些问题。<!-- more -->

## 常用解决方法
常用解决方法有两种

1. 获取`requestFactory`,然后设置超时。
2.  继承一个已有的`requestFactory`,构造RestTemplate时传入自己的`requestfactory`

第一种方式可以封装成如下代码:

```java
  public static void configTimeout(RestTemplate template, int readTimeout,
                                     int connectTimeout) {
        boolean updateReadTimeout = readTimeout >= 0;
        boolean updateConnectTimeout = connectTimeout >= 0;

        if (template == null || (!updateReadTimeout && !updateReadTimeout))
            return;
            ClientHttpRequestFactory factory = template.getRequestFactory();
            if (factory instanceof SimpleClientHttpRequestFactory) {
                SimpleClientHttpRequestFactory simple = (SimpleClientHttpRequestFactory) factory;
                if (updateReadTimeout)
                    simple.setReadTimeout(readTimeout);
                if (updateConnectTimeout)
                    simple.setConnectTimeout(connectTimeout);
            } else if (factory instanceof HttpComponentsClientHttpRequestFactory) {
                HttpComponentsClientHttpRequestFactory client = (HttpComponentsClientHttpRequestFactory) factory;
                if (updateReadTimeout)
                    client.setReadTimeout(readTimeout);
                if (updateConnectTimeout)
                    client.setConnectTimeout(connectTimeout);
            }
    }
```
如果使用aa框架，这样配置更麻烦，每次都需要获取`restTemplate`。如果使用aa框架，推荐使用第二种方式。
第二种方式，首先需要自定义一个类，继承一个`RequestFactory`,构造时设置超时。

```java
public class MyHttpFactory  extends SimpleClientHttpRequestFactory{

    public MyHttpFactory(){
        super();
        setConnectTimeout(7*1000);
        setReadTimeout(7*1000);

    }
```
然后构造restTemplate

```java
    public void test(){
        RestTemplate template = new RestTemplate(new MyHttpFactory());
        // 网络请求
    }
```

这种方式主要是在使用aa框架下使用，aa的rest接口可以传入requestFactory，我们自定义的factory就能用了,比如:

```java
@Rest(converters = {FormHttpMessageConverter.class, GsonHttpMessageConverter.class,
        StringHttpMessageConverter.class}, rootUrl = HttpURL.ROOT_URL + "tests/",
        requestFactory = MyHttpFactory.class)
public interface TestApi extends RestClientSupport {

    @Post("?name={name}")
    public void test(String name);
}

```

## 设置超时无效原因
在使用上述第一种解决方案时，添加了拦截器之后，发现设置超时无效，什么原因呢，可以从spring for android源码看看。
`RestTemplate`继承`InterceptingHttpAccessor`类，在`InterceptingHttpAccessor`类里实现了一个方法`getRequestFactory`,这个方法作用就是获取`RequestFactory`,方法实现如下

```java
	@Override
	public ClientHttpRequestFactory getRequestFactory() {
		ClientHttpRequestFactory delegate = super.getRequestFactory();
		if (!CollectionUtils.isEmpty(getInterceptors())) {
			return new InterceptingClientHttpRequestFactory(delegate, getInterceptors());
		}
		else {
			return delegate;
		}
	}

```
可以看到，如果`RestTemplate`有拦截器，则返回一个代理`ReqeustFactory`的`InterceptingClientHttpRequestFactory`，而`InterceptingClientHttpRequestFactory`并没有提显式提供什么方法获取真实的`RequestTemplate`。

## 超时无效的解决方法
从上例中方法我们可以看出解决方法，那就是先设置拦截器为空，然后设置超时，最后重新设置回拦截器。具体代码改动如下:

```java
  public static void configTimeout(RestTemplate template, int readTimeout,
                                     int connectTimeout) {
        boolean updateReadTimeout = readTimeout >= 0;
        boolean updateConnectTimeout = connectTimeout >= 0;

        if (template == null || (!updateReadTimeout && !updateReadTimeout))
            return;
        //这里是为了解决spring requestfactory不能获取到真实factory的问题，先设置过滤器为空，设置后超时后
        //再重新设置回去
        List<ClientHttpRequestInterceptor> interceptors = template.getInterceptors();
        template.setInterceptors(null);
        try {

            ClientHttpRequestFactory factory = template.getRequestFactory();
            //  InterceptingClientHttpRequestFactory interceptingClientHttpRequestFactory;


            if (factory instanceof SimpleClientHttpRequestFactory) {
                SimpleClientHttpRequestFactory simple = (SimpleClientHttpRequestFactory) factory;
                if (updateReadTimeout)
                    simple.setReadTimeout(readTimeout);
                if (updateConnectTimeout)
                    simple.setConnectTimeout(connectTimeout);
            } else if (factory instanceof HttpComponentsClientHttpRequestFactory) {
                HttpComponentsClientHttpRequestFactory client = (HttpComponentsClientHttpRequestFactory) factory;
                if (updateReadTimeout)
                    client.setReadTimeout(readTimeout);
                if (updateConnectTimeout)
                    client.setConnectTimeout(connectTimeout);
            }

        } finally {
            //重新设置回去
            template.setInterceptors(interceptors);
        }


    }
```
如果你觉得这种方式好麻烦，那就来个简单暴力点的，通过无所不能的反射来设置超时

```java
   /**
     * 配置RestTemplate超时时间,通过反射强制获取factry设置属性
     *
     * @param restTemplate
     * @param readTimeout    读取超时，如果小于等于0，则不设置
     * @param connectTimeout 连接超时时间，如果小于等于0，则不设置
     */
    public static void configTimeoutByReflect(RestTemplate restTemplate, int readTimeout, int connectTimeout) {
        ClientHttpRequestFactory factory = restTemplate.getRequestFactory();
        boolean updateReadTimeout = readTimeout >= 0;
        boolean updateConnectTimeout = connectTimeout >= 0;
        if (factory instanceof InterceptingClientHttpRequestFactory) {
            try {
                factory = (ClientHttpRequestFactory) ReflectUtils.getFieledValue(factory, "requestFactory");
                if(factory == null) return;
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }

        }


        if (factory instanceof SimpleClientHttpRequestFactory) {
            SimpleClientHttpRequestFactory simple = (SimpleClientHttpRequestFactory) factory;
            if (updateReadTimeout)
                simple.setReadTimeout(readTimeout);
            if (updateConnectTimeout)
                simple.setConnectTimeout(connectTimeout);
        } else if (factory instanceof HttpComponentsClientHttpRequestFactory) {
            HttpComponentsClientHttpRequestFactory client = (HttpComponentsClientHttpRequestFactory) factory;
            if (updateReadTimeout)
                client.setReadTimeout(readTimeout);
            if (updateConnectTimeout)
                client.setConnectTimeout(connectTimeout);
        }


    }

```
**ReflectUtils**代码如下:

```java
package com.dreamliner.secretchat.utils;

import java.lang.reflect.Field;

/**
 * Created by Administrator on 2015/5/12.
 */
public class ReflectUtils {


    /**
     * Gets fieled value.
     *获取到Filed的值
     * @param obj the obj
     * @param fieldName the field name
     * @return the fieled value
     * @throws IllegalAccessException the illegal access exception
     */
    public static Object getFieledValue(Object obj, String fieldName)
            throws IllegalAccessException {
        Field field = getFieldByRecursion(obj.getClass(), fieldName);
        if (field != null) {
            field.setAccessible(true);
            return field.get(obj);
        }
        return null;

    }


    /**
     * Sets field value.
     *
     * @param obj the obj
     * @param fildName the fild name
     * @param fieldValue the field value
     * @return the field value
     */
    public static boolean setFieldValue(Object obj, String fildName,
                                        Object fieldValue) {
        Field field = getFieldByRecursion(obj.getClass(), fildName);
        boolean result = false;
        if (field != null) {
            field.setAccessible(true);
            try {
                field.set(obj, fieldValue);

                result = true;
            } catch (IllegalArgumentException | IllegalAccessException e) {
                e.printStackTrace();
            }

        }
        return result;
    }

    /**
     * Gets field by recursion.
     *
     * @param classes the classes
     * @param fildName the fild name
     * @return the field by recursion
     */
// 用递归实现
    public static Field getFieldByRecursion(Class<?> classes, String fildName) {
        if (classes == null)
            return null;
        Field field = getFieldInCurrentClass(classes, fildName);

        return field == null ? getFieldByRecursion(classes.getSuperclass(), fildName)
                : field;

    }

    /**
     * Gets field by loop.
     *
     * @param classes the classes
     * @param fildName the fild name
     * @return the field by loop
     */
// 用循环实现
    public static Field getFieldByLoop(Class<?> classes, String fildName) {
        boolean havaField = false;
        Field field = null;
        Class<?> currentClass = classes;
        while (!havaField) {
            if (currentClass == null)
                break;

            field = getFieldInCurrentClass(currentClass, fildName);
            if (field != null)
                break;
            currentClass = currentClass.getSuperclass();

        }

        return field;

    }

    /**
     * Gets field in current class.
     *
     * @param classes the classes
     * @param fildName the fild name
     * @return the field in current class
     */
    public static Field getFieldInCurrentClass(Class<?> classes, String fildName) {
        Field[] declaredFields = classes.getDeclaredFields();
        for (Field field : declaredFields) {
            if (field.getName().equals(fildName)) {
                return field;
            }
        }
        return null;

    }


}

```





