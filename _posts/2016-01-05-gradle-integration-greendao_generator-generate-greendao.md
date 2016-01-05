---
layout: post
title: "gradle集成greendao-generator生成android端greendao"
description: "gradle integration greendao_generator generate greendao"
category: android
tags: [greendao,gradle]
date: 2016-01-05 16:30:40
---
以前使用greendao时，需要一个辅助的java项目，用于生成android端greendao代码。现在开发android项目基本是用android studio，构建工具使用gradle。gradle非常灵活强大，理论上讲，java能做的，gradle都能做。因此把`greendao-generator`集成在gradle中也不算什么难事。这样做的好处是，少了一个`java module`，省事。<!-- more -->
具体想法是在`build.gradle`中定义一个`gradle task`,在`task`里调用`greendao-generator`接口。

## 定义`greendaoGenerate` Task

在项目跟路径的`build.gradle`中:

```groovy
import de.greenrobot.daogenerator.Entity
import  de.greenrobot.daogenerator.Schema
import de.greenrobot.daogenerator.DaoGenerator

buildscript {
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        // replace with the current version of the Android plugin
        classpath 'com.android.tools.build:gradle:2.0.0-alpha3'
        // replace with the current version of the android-apt plugin
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'

        classpath   'de.greenrobot:greendao-generator:2.1.0'
    }
}

repositories {
    mavenCentral()
}

task greendaoGenerate <<{
    def pack = 'com.dreamliner.secretchat'
    Schema schema  = new Schema(1,"${pack}.entity");
    schema.defaultJavaPackageDao="${pack}.dao"
    schema.hasKeepSectionsByDefault=true
    schema.useActiveEntitiesByDefault=true;
    Entity user = schema.addEntity("User");
    user.addIdProperty();
    user.addStringProperty("name");
    user.addStringProperty("password");

    new DaoGenerator().generateAll(schema,"app/src/main/java",null,"app/src/test/java");

}
```
主要步骤是:

1. 在`buildScript`闭包中，定义仓库，然后添加依赖`de.greenrobot:greendao-generator:2.1.0`。这里的依赖是执行`gradle task`所需要的依赖。
2. `import`需要的包
3. 新增task(`greendaoGenerate`),在task中调用`DaoGenerator`。

## 生成dao
很简单，执行`greendaoGenerate` task 就行了

```shell
gradle greendaoGenerate
```
结果如下:
![gradle-greendao-result](http://7xivpo.com1.z0.glb.clouddn.com/gradle-greendao-resutl.png)




