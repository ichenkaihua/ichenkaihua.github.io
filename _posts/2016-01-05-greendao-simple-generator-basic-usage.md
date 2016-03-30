---
layout: post
title: "GreeDAO-Simple-Generate基本使用方法"
description: "Greendao Simple Generator basic usage"
category: android
tags: [Greendao Simple Generator,android orm]
date: 2016-01-05 01:49:09
permalink: /:year/:month/:day/:title/
---

[GreeDAO-Simple-Generate](https://github.com/ichenkaihua/GreenDAO-Simple-Generate),基于`de.greenrobot:greendao-generator`项目，运用少量java注解简化Greendao的生成难度。
这是我的第一个开源项目，源码托管在github，项目已经发布到[maven中心](http://mvnrepository.com/artifact/com.github.ichenkaihua/greendao-generate/0.0.3)，欢迎大家改进。<!-- more -->

## 引入

```groovy
    dependencies {
	    compile 'com.github.ichenkaihua:greendao-generate:0.0.4'
	}
    repositories {
	     mavenCentral()
	}
```

## 注解生成GreenDao

```java
package com.dreamliner.ckh.greendao;
	
	import com.github.ichenkaihua.greendao.annotation.EntityInject;
	import com.github.ichenkaihua.greendao.annotation.GenerateConfig;
	import com.github.ichenkaihua.greendao.annotation.SchemaConfig;
	import com.github.ichenkaihua.greendao.pojo.GenerateInfo;
	import com.github.ichenkaihua.greendao.pojo.SchemaInfo;
	import com.github.ichenkaihua.greendao.service.GenerateService;
	
	import de.greenrobot.daogenerator.Entity;
	import de.greenrobot.daogenerator.Schema;
	
	//GenerateConfig.outDir: 输出文件夹路径，必须存在这个路径，为避免出错，建议路径分隔符用 /
	//这个路径可以是绝对路径，/home/chenkaihua/workspace/QueryScore/src/main/java
	//也可以是相对路径，比如在同一个workspace的android项目名为 MyAndroidProject,则可以写成 ../MyAndroidProject/src
	//特别注意的是，如果是android studio项目，应该这样写 app/src/main/java
	// GenerateConfig.SchemaConfig.defaultJavaPackage 生成的entity的所在包，如果没有包名或包名不完全，则greendao会自动创建
	//GenerateConfig.SchemaConfig.defaultJavaPackageDao 生成的dao类所在包，如果没有包名或包名不完全，则greendao会自动创建
	@GenerateConfig(outDir = "/home/chenkaihua/workspace/QueryScore/src/main/java",
           schemaConfig = @SchemaConfig(defaultJavaPackage = "com.dreamlienr.queryscore.entity", 
          defaultJavaPackageDao = "com.dreamlienr.queryscore.dao"))
	public class Main {
	
		// 通过EntityInject注解，可以定义Entity的名字
		public void createUserEntity(@EntityInject("User") Entity user) {
			user.addIdProperty();
			user.addStringProperty("name");
			user.setSkipGeneration(true);
	
		}
	
		// 还可以定义Entity其他属性
		void createStudentEntity(
				@EntityInject(value = "Student", implementsSerializable = true, tableName = "MY_STUDENT") Entity student) {
			student.addIdProperty();
			student.addStringProperty("name");
	
		}
	
		// 可以注入多个Entity
		private void createMulitEntity(@EntityInject("Order") Entity order,
				@EntityInject("Photo") Entity photo) {
			order.addIdProperty();
			order.addStringProperty("name");
			photo.addIdProperty();
			photo.addStringProperty("path");
		}
	
		// 当然，你也可以注入de.greenrobot.daogenerator.Schema。
		// 也可以注入 GenerateInfo，GenerateInfo对象就是@GenerateConfig的注解信息
		// 还可以注入SchemaInfo ,SchemaInfo就是@SchemaConfig的信息
		void createMuiltEntityBySchema(Schema schema, GenerateInfo generateInfo,
				SchemaInfo schemaInfo) {
	
			System.out.println("Generate Info :" + generateInfo);
			System.out.println("Schema Info :" + schemaInfo);
			Entity dog = schema.addEntity("Dog");
			dog.addIdProperty();
			dog.addStringProperty("name");
	
			Entity cat = schema.addEntity("Cat");
			cat.addIdProperty();
			cat.addStringProperty("name");
	
		}
	
		// Main方法
		public static void main(String[] args) {
			// 传入带有类注解的Main.class类，GreenDao-Simple-Generate将会扫描类注解，配置输出路径，输出包名
			// 然后扫描这个类下的所有（public protected甚至private）方法
			// 如果方法参数有Schema、GenerateInfo、SchemaInfo、@EntityInject注解，则系统会注入相应对象
			// 最后调用这个方法
			new GenerateService(Main.class).generate();
		}
	
	}


	// Main方法
	public static void main(String[] args) {
		// 传入带有类注解的Main.class类，GreenDao-Simple-Generate将会扫描类注解，配置输出路径，输出包名
		// 然后扫描这个类下的所有（public protected甚至private）方法
		// 如果方法参数有Schema、GenerateInfo、SchemaInfo、@EntityInject注解，则系统会注入相应对象
		// 最后调用这个方法
		new GenerateService(Main.class).generate();
	}

```


## 注解详情
`GreeDAO-Simple-Generate`定义了三个注解:

* `@GenerateConfig`:类注解，用于定义输出配置。属性有:
	* `outDir`:定义输出目录-`String`类型，必填项。
	* `outDirTest`:	定义测试文件输出目录,`String`类型。
	* `outDirEntity`: 定义`Entity`java源文件输出目录,`String`类型。
	* `schemaConfig`:配置数据库信息,`com.github.ichenkaihua.greendao.annotation.SchemaConfig`类型，必填项。
* `@SchemaConfig`:配置数据库信息,属性有:
	* `version`: 定义版本,`int`类型，默认值为`1`。
	* `enableKeepSectionsByDefault`:是否默认保存`keep`区的代码，`boolean`型，默认为`true`。
	* `enableActiveEntitiesByDefault`:目前不知道作用,`String`类型。
	* `defaultJavaPackage`:默认的java包名，`String`类型，必填项。
	* `defaultJavaPackageTest`:默认输出测试文件的包名。
	* `defaultJavaPackageDao`: 默认输出`dao`java类的包名。
* `@EntityInject`:方法参数注解，注解一个`entity`,被注解的类型必须是`de.greenrobot.daogenerator.Entity`类型或者其子类。属性有：
	* `value`:entity的java类名，`String`类型，必填项。
	* `implementsSerializable`:是否让`entity`实现`java.io.Serializable`接口,`boolean`类型，默认为`false`。
	* `tableName`,表名，如果不填，则表名为java类名的大写字母，驼峰用`_`分开。`String`类型。

## 自动注入类型
处理上面的注解注入，`GreeDAO-Simple-Generate`支持对一下类型自动注入，即不需要用注解显示注入的类型:

* `de.greenrobot.daogenerator.Schema`:对应`greendao-genete`的`de.greenrobot.daogenerator.Schema`。
* `com.github.ichenkaihua.greendao.pojo.GenerateInfo`:对应`@GenerateConfig`注解属性。
* `com.github.ichenkaihua.greendao.pojo.SchemaInfo`:对应`@SchemaConfig`属性。

## 生成greendao代码

```java
public static void main(String[] args) {			
			new GenerateService(Main.class).generate();
		}
```
构造`GenerateService`时需要传入一个使用了`@GenerateConfig`注解的`class`。如果传入的`class`没有注解`@GenerateConfig`，则会报错。最后调用`generate()`方法即可生成android端greendao代码。
