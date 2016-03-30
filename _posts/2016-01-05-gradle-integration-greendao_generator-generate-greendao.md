---
layout: post
title: "gradle脚本集成greendao-generator生成android端greendao"
description: "gradle integration greendao_generator generate greendao"
category: android
tags: [greendao,gradle]
date: 2016-01-05 16:30:40
permalink: /:year/:month/:day/:title/
---
以前使用greendao时，需要一个辅助的java项目，用于生成android端greendao代码。现在开发android项目基本是用android studio，构建工具使用gradle。gradle非常灵活强大，理论上讲，java能做的，gradle都能做。因此把`greendao-generator`集成在gradle中也不算什么难事。这样做的好处是，少了一个`java module`，省事。<!-- more -->
具体想法是在`build.gradle`中定义一个`gradle task`,在`task`里调用`greendao-generator`接口。

## 添加greendao依赖
编辑打开`app/build.gradle`文件,添加android程序`greendao`依赖

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.2.1'

    //加入greendao的依赖
    compile 'de.greenrobot:greendao:2.1.0'
}
```



## 定义`greendaoGenerate` Task

在项目根路径的`build.gradle`中:

```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.
import de.greenrobot.daogenerator.Entity
import de.greenrobot.daogenerator.Schema
import de.greenrobot.daogenerator.DaoGenerator

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        //加入greendao-generator的依赖，gradle执行task的环境依赖在这里定义

        classpath 'com.android.tools.build:gradle:2.0.0-beta7'
        classpath 'de.greenrobot:greendao-generator:2.1.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

task greendaoGenerate << {
    def pack = 'com.chenkaihua.gradlegreendaosimple'
    Schema schema = new Schema(1, "${pack}.entity");
    schema.defaultJavaPackageDao = "${pack}.dao"
    schema.hasKeepSectionsByDefault = true
    schema.useActiveEntitiesByDefault = true;
    Entity user = schema.addEntity("User");
    user.addIdProperty();
    user.addStringProperty("name");
    user.addStringProperty("password");

    new DaoGenerator().generateAll(schema, "app/src/main/java", null, "app/src/test/java");

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
第一次运行时，gradle会下载`greendao-generator`依赖，速度可能比较慢
![gradle-greendao-download-jar](http://7xivpo.com1.z0.glb.clouddn.com/greendao-download_jar.png)

之后可以从控制台看到generate信息
![gradle-greendao-result](http://7xivpo.com1.z0.glb.clouddn.com/gradle-greendao-resutl.png)

完成之后刷新项目，是这个样子的

![gradle-greendao-app-view](http://7xivpo.com1.z0.glb.clouddn.com/greedao-result_view_2.png)

## Demo下载

Demo我已经传到github了,有兴趣的同学可以瞄一瞄

<https://github.com/ichenkaihua/GradleGreendaoSimple>



