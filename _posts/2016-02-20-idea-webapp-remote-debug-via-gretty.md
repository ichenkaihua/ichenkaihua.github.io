---
layout: post
title: "IntelliJ IDEA +gradle+gretty debug j2ee web-application"
description: "idea webapp remote  debug via gretty"
category: j2ee
tags: [debug,gretty]
date: 2016-02-20 12:47:44
permalink: /:year/:month/:day/:title/
---
前段时间被问到 如何在idea社区免费版IDE中debug  j2ee webapp项目。在IDEA付费版中，IDEA直接提供了tomcat jetty等插件，可以很方便的debug，然而在社区版并没有这些插件。但是我们可以通过结合`gretty`等`gradle`的webapp插件和IDEA的`Run/Debug Configurations`来实现`Remote Debug`。<!-- more -->

## IDEA版本
版本不限，免费版和社区版都行，比如我的
![idea](http://7xivpo.com1.z0.glb.clouddn.com/2016-02-20%2013%3A12%3A29%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

## 引入web-app项目到IntelliJ IDEA
本例使用[https://github.com/ichenkaihua/ssm-easy-template](https://github.com/ichenkaihua/ssm-easy-template)项目，项目中用了gradle 的`gretty`插件

## 添加`Remote Debug` Configuration

* 选择`Run/Edit Configurations`下拉菜单(右上角),点击`Edit Configurations...` ![](http://7xivpo.com1.z0.glb.clouddn.com/idea_run_debug_configuration.png)
* 在`Run/Edit Configurations`对话框中，单击**+** 图标，选择`Remote`,命名配置为`Remote Debug`（自己随便取）,什么都不用改，单击`OK`![](http://7xivpo.com1.z0.glb.clouddn.com/idea_run_debug_config_remote_debug.png)

## 设置断点
在代码中设置断点
![](http://7xivpo.com1.z0.glb.clouddn.com/break_point.png)

## debug web-app 项目
展开打开右侧`Gradle`选项窗口,点击刷新图标，之后如下
![](http://7xivpo.com1.z0.glb.clouddn.com/gradle_dia.png)

1.  执行`gradle appStartDebug`命令(双击`greety`下的`appStartDebug`),之后在右下角可以看到**Listening for transport dt_socket at address: 5005**等字样
![](http://7xivpo.com1.z0.glb.clouddn.com/debuging.png)
2. 窗口右上角点击选择`Remote Debug`，然后点击旁边的**debug**图标
![](http://7xivpo.com1.z0.glb.clouddn.com/remote_debug_debug.png)
接着看到**Connected to the target VM, address: 'localhost:5005', transport: 'socket'**字样
![](http://7xivpo.com1.z0.glb.clouddn.com/debuging_vm.png)
3. 在Run窗口可以看到项目正在启动（启动可能比较慢）
![](http://7xivpo.com1.z0.glb.clouddn.com/debug_run.png)
4.  在浏览器或者命令行中访问要debug的内容,我这里用`curl`命令
![](http://7xivpo.com1.z0.glb.clouddn.com/debug_curl.png)
5. 在IDEA的debug窗口中，可以看到如下，这就是配置成功了
![](http://7xivpo.com1.z0.glb.clouddn.com/debug_break.png)

## 停止debug

* 在`Debug`窗口中单击左侧红色的`Stop`图标，关闭debug
* 执行`gradle appStop`命令（在gradle窗口中gretty选项卡下双击`appStop`)

## 参考资料
* <https://github.com/akhikhl/gretty/issues/36>
