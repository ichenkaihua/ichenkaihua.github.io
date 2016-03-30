---
layout: post
title: "Ebean-ORM enhance with gradle"
description: "Ebean-ORM enhance with gradle"
category: j2ee
tags: [gradle,ebean-orm]
date: 2015-11-08 22:12:43
permalink: /:year/:month/:day/:title/
---


使用`Ebean ORM`有个麻烦的地方，就是每次部署app前，需要`enhance`下`entity`类的class文件，所谓`enhance`，就是加强操作，用于修改实体bean，包括"编织“，”转换“，”字节码操作“等过程。如果没有`enhance`就使用ebean orm，则ebean会抛出异常。`Ebean ORM`提供了eclipse插件、idea插件、maven插件、ant等解决方案，虽然没有gradle插件支持，不过好在gradle支持ant任务，通过gradle调用ebean提供的ant target，完成编译后自动`enhance`操作。<!-- more -->



### gradle调用ant

gradle调用ant的一般步骤是

1. 定义一个`configuration`配置组,就是一个依赖组，和`compile`,`testCompile`一个性质。
2. 给`configuration`定义依赖，ant运行时需要依赖这些包。
3. 使用`ant.taskdef`闭包定义一个ant task。
4. 定义一个task，调用步骤3定义的ant。

**build.gradle**文件:

```groovy
repositories {
    mavenCentral()
}

configurations{
    ebeanagent
}

dependencies {
    ebeanagent 'org.avaje.ebeanorm:avaje-ebeanorm-agent:4.7.1'
}

ant.taskdef(
        name: "ebeanEnhance",
        classname:"com.avaje.ebean.enhance.ant.AntEnhanceTask",
        classpath:configurations.ebeanagent.asPath

)

def ebeanEnhance = {dir, packages ->
    println dir
    println packages
    println '============================================'
    println '  Enhance ebean classes....' + dir
    println '============================================'
    ant.ebeanEnhance(classSource: dir,
            packages: packages,
            transformArgs: "debug=10")
    println 'Enhance ebean end....................'
}



compileJava.doLast {
    ebeanEnhance(destinationDir, "com.chenkaihua.springebean.entity.*")
}

```
 这里没有定义task调用ant，而是让`compileJava`之后`enhance`，这样就保证了自动`enhance`，相比idea插件等手动`enhance`方式，省事多了。

### 小技巧
使用**IDEA付费版**时，跑server用idea集成工具，比如tomcat，jetty等，此时按照上述解决办法`enhance`后还是不能跑，报错为**com.xxx.entity.User not enhance?**,说明entity没有`enhance`，解决方法如下：

1.**打开run configurations**:
![idea-run-configuration](http://7xivpo.com1.z0.glb.clouddn.com/idea-run-config.png)
2.**增加build前的gradle task为'compileJava'**
![idea-run-config-view](http://7xivpo.com1.z0.glb.clouddn.com/idea-run-config-view.png)
![idea-run-config-view-add-task-pop](http://7xivpo.com1.z0.glb.clouddn.com/idea-run-config-view-add-task-pop.png)
![idea-run-config-select-gradle-task](http://7xivpo.com1.z0.glb.clouddn.com/idea-run-config-select-gradle-task.png)
3. **调整顺序**：
![idea-run-config-final](http://7xivpo.com1.z0.glb.clouddn.com/idea-run-config-final.png)

如果是使用`gretty`，`jetty`等插件部署app，则不需要add这个task。

### 参考资料
>
* [Ebean 3 应用纯心理感受 -  郁也风的个人空间 - 开源中国社区][1]
 
[1]: http://my.oschina.net/someok/blog/184078
