---
layout: post
title: "在gradle中使用MyBatis Generator生成model,dao,mapper"
description: "running mybatis generator with gradle"
category: j2ee
tags: [mybatis,mybatis-generator,gradle]
date: 2015-12-19 17:13:12
permalink: /:year/:month/:day/:title/
---
**[Mybatis Generator][1]**是一个mybatis工具项目，用于生成mybatis的`model`,`mapper`,`dao`持久层代码。**Mybatis Generator**提供了`maven plugin`,`ant target`，`java`三种方式启动。现在主流的构建工具是**gradle**,虽然mybatis generator没有提供gradle的插件，但gradle可以调用ant任务，因此，gradle也能启动Mybatis Generator。<!-- more -->

## 环境说明

* 数据库:`mysql`
* 数据库配置文件:`src/main/resources/db-mysql.properties`
* 项目中使用了mybatis`tk.mybatis:mapper:3.3.2`插件


## 配置依赖
在**build.gradle**中:
运行ant需要运行环境，也就是相应的jar包，因此添加一个配置

```groovy
configurations {

    mybatisGenerator
}

```
给这个配置添加依赖

```groovy
repositories {
    mavenCentral()
}
dependencies {
    mybatisGenerator 'org.mybatis.generator:mybatis-generator-core:1.3.2'
    mybatisGenerator 'mysql:mysql-connector-java:5.1.36'
    mybatisGenerator 'tk.mybatis:mapper:3.3.2'
}
```

## 配置task
在**build.gradle**中，定义一个task，在task里面调用ant

```groovy

def getDbProperties = {
    def properties = new Properties()
    file("src/main/resources/db-mysql.properties").withInputStream { inputStream ->
        properties.load(inputStream)
    }

    properties;

}


task mybatisGenerate << {


    def properties = getDbProperties()



    ant.properties['targetProject'] = projectDir.path
    ant.properties['driverClass'] = properties.getProperty("jdbc.driverClassName")
    ant.properties['connectionURL'] = properties.getProperty("jdbc.url")
    ant.properties['userId'] = properties.getProperty("jdbc.user")
    ant.properties['password'] = properties.getProperty("jdbc.pass")
    ant.properties['src_main_java'] = sourceSets.main.java.srcDirs[0].path
    ant.properties['src_main_resources'] = sourceSets.main.resources.srcDirs[0].path
    ant.properties['modelPackage'] = this.modelPackage
    ant.properties['mapperPackage'] = this.mapperPackage
    ant.properties['sqlMapperPackage'] = this.sqlMapperPackage

    ant.taskdef(
            name: 'mbgenerator',
            classname: 'org.mybatis.generator.ant.GeneratorAntTask',
            classpath: configurations.mybatisGenerator.asPath
    )
    ant.mbgenerator(overwrite: true,
            configfile: 'db/generatorConfig.xml', verbose: true) {
        propertyset {
            propertyref(name: 'targetProject')
            propertyref(name: 'userId')
            propertyref(name: 'driverClass')
            propertyref(name: 'connectionURL')
            propertyref(name: 'password')
            propertyref(name: 'src_main_java')
            propertyref(name: 'src_main_resources')
            propertyref(name: 'modelPackage')
            propertyref(name: 'mapperPackage')
            propertyref(name: 'sqlMapperPackage')

        }
    }
}
```
大致思路是:

* 从`db-mysql.propertis`读取配置
*  把配置注入`ant`任务
* 运行`ant`,生成文件

数据库配置文件，文件在`src/main/resources/db-mysql.properties`

```groovy
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost/mhml
jdbc.user=root
jdbc.pass=rootpass
```
其他配置,`gradle.propertis`

```
modelPackage=com.dreamliner.mhml.entity
#生成的mapper接口类所在包
mapperPackage=com.dreamliner.mhml.mapper
#生成的mapper xml文件所在包，默认存储在resources目录下
sqlMapperPackage=mybatis_mapper
```


## db/generatorConfig配置

`db/generatorConfig.xml`文件用于配置生成策略,[官网详细介绍][2]

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">


        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <!-- caseSensitive默认false，当数据库表名区分大小写时，可以将该属性设置为true -->
            <property name="caseSensitive" value="true"/>
        </plugin>


        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="${driverClass}"
                        connectionURL="${connectionURL}"
                        userId="${userId}"
                        password="${password}">
        </jdbcConnection>


        <javaModelGenerator targetPackage="${modelPackage}" targetProject="${src_main_java}"/>

        <sqlMapGenerator targetPackage="${sqlMapperPackage}" targetProject="${src_main_resources}"/>

        <javaClientGenerator targetPackage="${mapperPackage}" targetProject="${src_main_java}" type="XMLMAPPER"/>
      <!-- sql占位符，表示所有的表 -->
	    <table tableName="%">
                    <generatedKey column="epa_id" sqlStatement="Mysql" identity="true" />
                </table>

    </context>
</generatorConfiguration>
```

## run,生成代码

```shell
gradle  mybatisGenerate
```


[1]: http://mybatis.org/generator/index.html
[2]: https://github.com/abel533/Mapper








