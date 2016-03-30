---
layout: post
title: "解决eclipse刷新maven项目,jdk变为1.5问题"
description: "jdk version problem with maven"
category: [build tools]
fullwidth: true
tags: [maven,jdk]
date: 2016-02-24 23:00:40
permalink: /:year/:month/:day/:title/
---
想起了以前在eclipse里使用maven构建j2ee项目时，困扰我一天的问题。即默认新建完的maven项目jdk版本为1.5(不支持一些注解)，因此我右键修改为安装版本,这时没有问题，但是使用了`maven update`或者刷新项目后，jdk版本又变为1.5版本，如此反复。开始以为是项目问题，网上找资料才发现是maven配置的问题,解决这个问题需要修改maven的配置文件。<!-- more -->

maven配置的问题就不说了，好多教程。现在不用eclipse，只记得大概。


1. 首先找到maven的配置文件`settings.xml`，通常在`maven_home/conf`目录下
2. 在` <profiles> </profiles>`节点中添加如下配置,把`1.7`改成安装的jdk版本,然后刷新项目。

```xml
<profile>     
            <id>jdk-1.7</id>     
            <activation>     
                <activeByDefault>true</activeByDefault>     
                <jdk>1.7</jdk>     
            </activation>     
            <properties>     
                <maven.compiler.source>1.7</maven.compiler.source>     
                <maven.compiler.target>1.7</maven.compiler.target>     
                <maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>     
            </properties>     			
    </profile>  
```
