---
layout: post
title: "how to use greendao"
description: "如何使用greendao| 陈开华博客"
category: android
tags: [greendao,android-orm]
date: 2015-11-19 23:13:44
permalink: /:year/:month/:day/:title/
---
开发**android**应用时，免不了和**sqlite**数据库打交道。如果通过android源生api操作数据库，不但费时费力，使得后期维护变得困难，而且不能保证有良好的性能表现。许多热心的开源组织或个人，致力于解决这个难题，帮助开发者用最少的时间开发出高性能的sqlite应用。**greendao**就是这样的开源项目,在**android-orm**类项目中，使用人数最多。<!-- more -->

## 介绍
> greendao是一个开源项目，帮助android开发者快速开发高性能的sqlite应用。众所周知，sqlite是一个嵌入式关系型数据库。如果去开发sqlite应用，需要做大量的额外工作，需要写sql语句并且把查询结果转换成java对象等非常多步骤。
greendao能帮助你这些：他映射java对象和数据库中的表，并且提供方便的api供你调用。这样就可以让你的增删改查操作直接面向java对象，而不用拼凑容易出错的sql语句，帮助你把时间用在解决真正需要解决的问题上。

### 模块介绍
greendao由两个子项目组成:

* `de.greenrobot:greendao-generator`:用于java项目。作用是在java项目里配置android项目数据库信息、表信息、包名、最后在指定的路径下，生成指定的包名类名的android代码。也就是说，这个项目用于生成android代码。
* `de.greenrobot:greendao` :用于android项目。上述java生成的android代码有些基类，工具类，都在这个jar包里提供，如果android项目没有依赖这个包，生成的android代码会报错。也就是说，这个项目提供了android代码的依赖。

## 使用
使用greendao一般需要以下几个步骤:

1. 新建android项目，依赖**de.greenrobot:greendao**。比如项目名为`app`
2. 新建java项目，依赖**de.greenrobot.greendao-generator**。例如项目名叫`greendao-generate`
3. java项目配置好数据库信息，设置好生成路径等信息。
4. 运行java项目
5. 刷新android项目，根据greendao接口开发应用。



### java项目-greendao-generate的使用



### android项目-greendao的使用








[ormlite]: http://ormlite.com/sqlite_java_android_orm.shtml
[greendao]: http://greendao-orm.com/