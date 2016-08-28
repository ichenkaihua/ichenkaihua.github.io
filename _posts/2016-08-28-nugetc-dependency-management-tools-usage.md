---
layout: post
title: "c#项目依赖管理工具 nuget的使用和项目上传"
description: "nuget:c# Dependency management tools usage"
category: [c#]
tags: [nuget]
---

写过java的应该都知道，java有许多依赖管理工具，使用最多的是**gradle**和**maven**。这些工具解决了项目依赖问题，添加或者去除依赖只需要一条语句而无需改动jar包。c#下也有类似的工具，那就是:[nuget](https://www.nuget.org/)。尽管nuget没有gradle和maven强大，但总比手动引入dll方便。<!-- more -->

## nuget简介

项目官网:<https://www.nuget.org/>

以下介绍来自官网:

> ### What is NuGet?
>> NuGet is the package manager for the Microsoft development platform including .NET. The NuGet client tools provide the ability to produce and consume packages. The NuGet Gallery is the central package repository used by all package authors and consumers. 

简单来说:

* nuget是.net平台下的包管理器。
* nuget提供了工具集成，以便程序可以使用nuget仓库上的包，并且有工具将项目打包上传到nuget仓库，供别人使用。

## visual studio上安装nuget

我使用的`visual studio 2015`自带了nuget，如果其他版本没有，可以到[这里](http://dist.nuget.org/index.html)下载`vsix`文件安装扩展。

## 使用nuget添加依赖

可以使用nuget的可视图形化操作或者命令行操作来添加依赖:

* 可视图形化操作:
	* 菜单-->工具-->Nuget包管理器-->管理解决方案的NuGet程序包,![nuget_gui](http://7xivpo.com1.z0.glb.clouddn.com/nuget_gui.png)
	* 选择包后右边选择本地项目再点击安装就可以了
* 命令行:
	* 菜单-->工具-->Nuget包管理器-->程序包管理器控制台 ![nuget_cmd](http://7xivpo.com1.z0.glb.clouddn.com/nuget_cmd.png)
	* 安装命令为`Install-Package`,比如安装EasyHttp4Net.Core的命令为:`Install-Package EasyHttp4Net.Core`

总的来说，添加依赖非常简单。

## 上传项目到nuget

在完成一个库之后，觉得还不错。怎么让别人方便的获取到你的项目并引入到项目中去呢。那当然是使用nuget了，nuget上传库还是挺方便的。

### 配置并生成项目nuget包

* 下载[NuGet.exe](https://dist.nuget.org/win-x86-commandline/latest/nuget.exe)
* 将`nuget.exe`路径添加到系统环境变量`PATH`,因为后面要使用nuget的命令。
* 打开cmd命令行，并切换到项目所在路径,输入命令并回车:`nuget spec` 。
* 可以看到项目路径下生成了`xxx.nuspec`文件`xxx`是项目名。

`nuspec`文件默认如下：

```xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>Package</id>
    <version>1.0.0</version>
    <authors>Administrator</authors>
    <owners>Administrator</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>Package description</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2016</copyright>
    <tags>Tag1 Tag2</tags>
    <dependencies>
      <dependency id="SampleDependency" version="1.0" />
    </dependencies>
  </metadata>
</package>
```

根据情况修改,比如版本号，包名，项目介绍，项目链接等。这里是[详细配置](https://docs.nuget.org/Create/Creating-and-Publishing-a-Package)

#### 打包
在项目路径下打开cmd并输入命令：

```shell
nuget pack MyProject.csproj
```

可以看到项目文件夹下生成了`xxxx.nupkg`，这个`nupkg`文件就是等下要上传到nuget仓库的文件。


### 上传到nuget仓库中心

* 注册nuget帐号，推荐使用微软帐号关联
* 登录帐号
* 打开<https://www.nuget.org/packages/manage/upload>页面 ![nuget_upload_web](http://7xivpo.com1.z0.glb.clouddn.com/nuget_upload_web.png)
* 点击**选择文件**，选择刚刚生成的`nupkg`文件 ![nuget_upload_select_file](http://7xivpo.com1.z0.glb.clouddn.com/nuget_upload_select_file.png)
* 点击**upload**上传
* 确认后点击submit

## 实验项目

<https://github.com/ichenkaihua/EasyHttp4Net>

## 参考资料

> nuget:<https://www.nuget.org/>
> nuget-Documentation:<https://docs.nuget.org/>
