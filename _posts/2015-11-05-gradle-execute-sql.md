---
layout: post
title: "gradle执行sql初始化数据库"
description: "gradle执行sql初始化数据库-陈开华博客"
category: java
tags: [gradle,sql]
date: 2015-11-05 20:07:51
permalink: /:year/:month/:day/:title/
---


gradle是目前java应用最强大的自动化构建工具。gradle以groovy语言基础，基于DSL（领域特定语言)语法。因为基于groovy，所以java能做的事情，gradle都能做。由于gradle基于DSL语法，因此在配置gradle时，非常简洁灵活。
上面说了，gradle基于groovy语言，groovy又基于java，因此gradle无所不能。项目开发时，要在本地环境调试应用，涉及到数据库的初始化等步骤，技术难度不大，却要花费些时间。gradle完全可以帮助我们初始化数据库。<!-- more -->

#### 新建gradle目,直接使用`gradle init`命令：

```shell
mkdir project_demo
cd project_demo
# 生成build.gradle,settiongs.gradle等信息
gradle init
```

#### 配置`build.gradle`

1.定义一个`configuration`。所谓`configuration`，就是一个依赖组组名，`compile` `testCompile`就是两个`configuration`。比如定义名字为`driver`的`configuration`:

``` groovy
configurations {
    driver
}
```
2.给`driver`配置依赖,，使用mysql数据库。

```
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.11'
   driver 'mysql:mysql-connector-java:5.1.37'

}

```

3.定义task，执行初始化任务

```java
import groovy.sql.Sql;

task loadMySqlDriver << {
    description='load Mysql Driver Class by ClassLoader'
    URLClassLoader loader = GroovyObject.class.classLoader
    configurations.driver.each { File file ->
        loader.addURL(file.toURL())
    }
}

Sql getSql() {
    def props = [user: "github", password: "github-pass", allowMultiQueries: 'true'] as Properties
    def url = "jdbc:mysql://localhost:3306"
    def driver = "com.mysql.jdbc.Driver"
    Sql.newInstance(url, props, driver)


}

task initDB(dependsOn: loadMySqlDriver) << {
    description='get file text from sql/backup.sql and import to mysql'
    Sql sql = getSql()
    def text =  file("sql/backup.sql").text
    sql.execute(text)
}

```
>  **注意**:上述代码有个`loadMySqlDriver`的task，作用是载入mysql驱动文件，这可能是gradle的bug，如果不显式的载入，会抛出`ClassNotFoundException`。

下面是`sql/backup.sql`文件:

```sql
CREATE DATABASE  IF NOT EXISTS `demo` /*!40100 DEFAULT CHARACTER SET utf8 */;
USE `demo`;

DROP TABLE IF EXISTS `user`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `user` (
 `id` int(11) NOT NULL AUTO_INCREMENT,
 `name` char(40) DEFAULT NULL,
 `password` char(50) DEFAULT NULL,
 PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;

INSERT INTO user VALUES(null,'chenkaihua','ckh-pass')
```

完成所有配置的`build.gradle`代码如下:

```java
import groovy.sql.Sql;

group 'com.chenkaihua'
version '1.0-SNAPSHOT'


apply plugin: 'java'


configurations {
    driver
}


repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.11'
    driver 'mysql:mysql-connector-java:5.1.37'


}

task loadMySqlDriver << {
    description='load Mysql Driver Class by ClassLoader'
    URLClassLoader loader = GroovyObject.class.classLoader
    configurations.driver.each { File file ->
        loader.addURL(file.toURL())
    }
}

Sql getSql() {
    def props = [user: "github", password: "github-pass", allowMultiQueries: 'true'] as Properties
    def url = "jdbc:mysql://localhost:3306"
    def driver = "com.mysql.jdbc.Driver"
    Sql.newInstance(url, props, driver)


}

task initDB(dependsOn: loadMySqlDriver) << {
    description='get file text from sql/backup.sql and import to mysql'
    Sql sql = getSql()
    def text =  file("sql/backup.sql").text
    sql.execute(text)
}

```

#### 初始化数据库

接下来就是执行`initDB`这个任务,用于初始化数据库:

```
gradle initDB
```

