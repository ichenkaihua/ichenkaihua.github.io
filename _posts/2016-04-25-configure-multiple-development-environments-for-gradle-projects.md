---
layout: post
title: "为gradle项目配置多种开发环境"
description: "为gradle项目配置多种开发环境|configure multiple development environments for gradle projects"
category: [build tools]
tags: [gradle,multiple project]
---
项目开发中，通常有多个环境，一个是用于本地调试的开发环境，另一个是用于对外发布的生产环境。 在本地开发环境中，使用本地安装的数据库，在生产环境中使用生产环境的数据库。这样就能避免本地调试不当导致对生产环境数据造成破坏.使用gradle可以轻松配置多个开发环境，简单配置之后，用户只需修改一项配置文件即可切换数据库环境。<!-- more -->

## 思路
大致思路如下:

1. 在`src/main/`下建立多个资源文件夹,用于区别不同环境下的配置信息，比如 `src/main/resources-dev`,`src/main/resources-prod`
2. gradle读取`gradle.properties`确定使用哪种环境
3. gradle根据项目环境自动引入相应环境的配置文件,比如在dev环境下，引入`src/main/resources-dev`,在prod环境下，则引入`src/main/resources-prod`

**下面看具体实现**

## `gradle.properties`定义开发环境
在`gradle.properties`文件中，定义一个配置项。gradle启动时会自动读取这个文件，在项目的`build.gradle`可以直接获取这个属性

```
# 配置开发环境，可选为dev 或 prod,
#   如果为dev,则读取src/main/rescource-dev/jdbc-mysql.properties
#   如果为prod,则读取src/main/resources-prod/jdbc-mysql.properties
#   默认为dev
env=dev
```

## 配置`build.gradle`
在`build.gradle`中定义`sourceSets`,根据`env`的值选择`resource`,最后提供了一个闭包读取`jdbc-mysql.properties`的值，通常在数据库配置的task里调用这个闭包，使用`mybatis-generator`,`flywaydb`等插件时需要数据库信息，调用这个闭包就可以获取了。

```groovy
ext {
    if (!project.hasProperty("env")) {
        println '没有配置数据环境，默认使用 开发环境'
        env = "dev"
    }
    println "使用数据库环境为:${project['env']}"
}

sourceSets {
    main {
        resources {
            srcDir("src/main/resources")
            if (project['env'] == 'dev') {
                srcDir("src/main/resources-dev")
            } else if (project['env'] == 'prod') {
                srcDir('src/main/resources-prod')
            }
        }
    }
}

def getDbProperties = {
    def properties = new Properties()
    def dbPropertiesPath = sourceSets.main.resources.srcDirs[1].path;
    file("$dbPropertiesPath/jdbc-mysql.properties").withInputStream { inputStream ->
        properties.load(inputStream)
    }
    properties;
}

```

在spring等项目中就可以使用了:

```xml
<context:property-placeholder
        location="classpath:jdbc-mysql.properties" />
```

## demo
此功能已经集成到了[ssm-easy-template](https://github.com/ichenkaihua/ssm-easy-template)，欢迎下载。



